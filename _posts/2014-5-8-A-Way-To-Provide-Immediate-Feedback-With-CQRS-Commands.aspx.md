---
layout: post
title: A Way To Provide Immediate Feedback In CQRS
category: CQRS
---

Well, it's not just for CQRS, this post also applies for cases when you need to provide quick feedback for an action that should be valid from the business point of view (all relevant business rules must be valid) and you want to be DRY.

 We have the famous Order but this time we want to cancel it. Of course, there are some business rules that decide when/if an order can be cancelled, all beautifully encapsulated in a domain entity or service. When we design and implement that object we care only about the domain and not about other details like where this operation would be executed (in the same thread/process/server). After all when using message driven approach, chances are that a lot of the commands are executed by a background service who knows when.

 So, for now our service interface it looks like this


```csharp
public interface ICancelOrders
  {
        void Cancel(Order order);
  }
```
  If the order can't be cancelled then what? We may opt for a simple return (nobody will know about it), we can return a bool (or a result type) or we can throw an exception. I say it's better to go for the exception because it shouldn't get to that point unless it's a bug or something happened. Sure, we can return some status but it will be useless if the service is executed on a background thread/worker or on another server. And the domain shouldn't care about where its code executes.

 So we need to make sure that form the business point of view the cancelling is doable, if not, the user should be notified in that very moment. The easiest,cleanest way is to simply ask the service doing the job if it can really do it (same for a entity). The point is we make preparation to ensure proper domain behaviour. This means our service interface now looks like this


```csharp
public interface ICancelOrders
  {
        bool CanCancel(Order order);
	void Cancel(Order order);
  }
```
  Now, I understand that the service may execute 10 minutes after the user pressed the button, but the important thing is to ensure that the order can be canceled in the moment the user pressed that button. After all, cancelling may take be a long running process and lots of things can change, but in that moment in time, the user could cancel it. A valid behaviour. I'll talk later about this particular case.

 So, we make sure that the cancelling command can be executed at before it reaches the handler. But now you have another problem, the browser sends you a guid and your domain service knows only about domain objects. You can inject the service and a repository in the controller, get the Order, check if it can be canceled and add an error message if it can't, but this is out of the controller responsibility.

 Instead, we'll implement that functionality into an application service , something like this


```csharp
public class OrderCommandsService
 {
     public OrderCommandsService(IOrdersRepository repo, ICancelOrder service,IDispatchMessages bus) { /*.. */}

	 public bool CancelOrder(Guid id)
	 {
	    var order=_repo.Get(id);
		if (order==null) throw new OrderNotFound();//this might be a bug
		if (_service.CanCancel(order))
		{
			var cmd=new CancelOrder(){OrderId=id};
			_bus.Send(cmd);
			return true;
		}
		return false;
	 }
 }
```
  The controller will get the result of_ CancelOrder(id)_ and adds an error message or not. Now the user gets immediate feedback before the cancelling command is issued and we kept the business rules where they belong. But what if the command is executed 3 minutes later and the order is not cancelable anymore? The command handler catches the exception (it should be an exceptional case) and the **OrderCancelationFailed** domain event is published. And an event handler can notify the user that something happened and the Order couldn't be cancelled. It's up to the business to decide what happens next.

 After all, the customer could have tried to cancel the order by phone and the customer representative would have recorded the cancelation request but by the time conversation was over, the command wouldn't have been valid. It's the same outcome in the end, only the medium of submitting the request is different.

 The Domain should model in code the way the business handles these things and for some behaviour we can offer immediate feedback while for other it will be a later feedback. Not everything must be instant, let the business tell you which is which. Same for consistency.

 P.S: Keep all the business rules in the Domain and use application services to act as a bridge between the UI and the Domain. The UI will know only about the services, these will be the M in MVC.
