---
layout: post
title:  CQ(R)S And Providing Immediate Feedback In A Web App Using A Service Bus
category: Domain driven design
---

Whenever I'm developing a DDD web app, I'm using CQRS and Domain Events and this means I have a lot of commands and event handlers. Therefore I'm using a durable service bus which means that if the server crashes, all the unhandled messages are sent again to be handled. But I have a problem: all my commands are handled asynchronously because this is how my service bus works. And it works this way because a message can be handled anywhere in that process (usually in a background thread) or in another process if we have a distributed app.

Asynchrony means I can only 'send' a command but I can't expect an answer. Basically it's only

```
 bus.Send(new DoSomething(){});
```

and this sucks when application services are implemented as command handlers. Because the user expects some feedback. If the command fails you want to return some error and this is quite tricky in this scenario. The usual solution is to do some client side server polling checking the state of the operation or waiting for a read model. This is obviously cumbersome.

However, we can apply a similar concept but on server side using a mediator. Simply put, we'll have a an object in charge of delivering the command result to our controller. It goes like this in the controller

```csharp

//constructor
public MyController(IServiceBus bus,ICommandResultMediator mediator) {}


//action
var cmd=new DoSomething();

await _bus.SendAsync(cmd);

var listener = _mediator.GetListener(cmd.Id);

try
 {
     var result = await listener.GetResult<CommandResult>(cancel);

     return Response.AsJson(result);
 }
 catch (TimeoutException)
 {
     return HttpStatusCode.RequestTimeout;
 }

 catch (OperationCanceledException)
 {
     return HttpStatusCode.InternalServerError;
 }

```

This is NancyFx code but it's easy to understand, instead of returning an ActionResult or directly the model, I'm returning a NancyResponse which can be a HttpStatusCode as well. This is not important, the important part is the use of the mediator. We send the command to be handled then we ask the mediator for a listener that will be used to await the command result. If the result doesn't arrive in a couple of seconds or if the operation is cancelled, exceptions are thrown.

Now here's how the command handler looks like

```csharp
class DoSomethingHandler:IExecute<DoSomething>
{
  public DoSomethingHandler(ICommandResultMediator mediator) { }

  public void Execute(DoSomething cmd)
  {
    //do stuff
    //get result
    var result=new CommandResult();
    result.AddError("stuff","Stuff happened");
    _mediator.AddResult(cmd.Id,result);
  }
}

```

In the handler we do things as usual, the only change is that now the result is sent to the mediator. And here is the mediator itself

```csharp
public class CommandResultMediator : ICommandResultMediator
   {

       public CommandResultMediator()
       {
           ResultCheckPeriod = 100;
       }
       public void AddResult<T>(Guid cmdId, T result) where T :class
       {
           Listener l = null;
           if (_items.TryRemove(cmdId,out l))
           {
               l.Result = result;
           }

       }

       /// <summary>
       /// How often to check if a result has arrived, in ms.
       /// Default is 100 ms
       /// </summary>
       public int ResultCheckPeriod { get; set; }

       void Remove(Guid cmdId)
       {
            Listener l = null;
           _items.TryRemove(cmdId, out l);
       }

       public int ActiveListeners
       {
           get { return _items.Count; }
       }

       ConcurrentDictionary<Guid,Listener> _items=new ConcurrentDictionary<Guid, Listener>();

       public IResultListener GetListener(Guid cmdId,TimeSpan? timeout=null)
       {
           timeout = timeout ?? TimeSpan.FromSeconds(5);
           var listener=new Listener(timeout.Value,cmdId,this);
           listener.UpdatePeriod = ResultCheckPeriod;
           _items.TryAdd(cmdId, listener);
           return listener;
       }


       class Listener:IResultListener
       {
           private readonly Guid _cmdId;
           private readonly CommandResultMediator _parent;

           public Listener(TimeSpan timeout, Guid cmdId, CommandResultMediator parent)
           {
               _cmdId = cmdId;
               _parent = parent;
               TimeoutTime = DateTime.Now.Add(timeout);
           }

           private DateTime TimeoutTime { get; set; }

           public object Result { get; set; }

           public int UpdatePeriod=100;

           public async Task<T> GetResult<T>(CancellationToken cancel=default(CancellationToken)) where T : class
           {
               while(Result==null)
               {
                   if (DateTime.Now >= TimeoutTime)
                   {
                       _parent.Remove(_cmdId);
                       throw new TimeoutException();
                   }
                   if (cancel.IsCancellationRequested)
                   {
                       _parent.Remove(_cmdId);
                       throw new OperationCanceledException();
                   }
                   await Task.Delay(UpdatePeriod,cancel).ConfigureAwait(false);
               }
               return Result as T;
           }
       }
   }
```

Btw, the mediator should be a singleton managed by your favourite DI Container.

Note that the mediator is part of my [CavemanTools](https://www.nuget.org/packages/CavemanTools/) library so if you're using it, or [SqlFu](https://github.com/sapiens/SqlFu) or [MvcPowertools](https://github.com/sapiens/MvcPowerTools) you only need to update to the last version.

So pretty much that's it! The easiest way to return a result from an async command handler.
