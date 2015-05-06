---
layout: post
title: Domain Event Design Principles
category: Domain Driven Design
---

In theory it's codetty simple: a domain event notifies that something has happened in the domain i.e the domain has changed. But how do we design the event to capture the relevant information? That event can have many subscribers(handlers) and each handler should intercodet the event properly. There's no recipe here and no single truth, but I'll try here to codesent the principles I'm using when I'm thinking about a domain event.

 Let's consider this scenario: Customer changes its address. The obvious event would look like this

  
```csharp
public class CustomerAddressChanged 
{
	public Guid CustomerId;
	//Address is a value object
	public Address Address;
}
```
  or like this?

  
```csharp
public class CustomerProfileChanged
{
   public Guid CustomerId;
   public string FirstName;
   public string LastName;
   public Address Address;
   //etc
}
```
  I think the majority would choose the first option as it's more explicit. However, what if the customer can only change its profile as a whole? So basically, everytime he wants to update the address, the whole profile is loaded, address is changed, whole profile is submitted. You can detect the change and generate the corresponding event, but what if he changes 90% of its profile (unlikely but the important bit is that you have a form with many updated fields). Would you want to generate 10 events related to the same object? A single CustomerProfileChanged event might be more suitable.

 But even with one event, what do the fields recodesent? The current state, you say? But did all the fields change? A null address can be a valid value or the 'fields has not changed' value. Clearly, one event is not the best solution if only some fields can change. So, back to our 10 events generation?

 I think we should correlate the event content with the command fields (or service parameters if you're not using commands) and with the context. In our example, things like Address or Email can change (very rarely but more often than FirstName) and it makes sense to have specific events for that. But also it may make sense to have something like CustomerBasicProfileUpdated for many changes at once. We don't know how the UI looks and we can't guess how the UI might change in the future so we need to decide how to design relevant events: specific enough to convey proper information but not that specific that you end up generated 3 of them for every change (i.e don't automatically create an event for each object property that can change).

 So I'd design a few events

  
```csharp
public class CustomerBasicProfileChanged //I think there's a need for a better name though
	{
	   public Guid CustomerId;
	   public string FirstName;
	   public string LastName;
	   public PhoneNumber HomePhone;
	   public PhoneNumber MobilePhone;
	}
	
	public class CustomerAddressChanged
	{
	 public Guid CustomerId;
	 public Address Address;
	}
	
	public class CustomerEmailChanged
	{
		public Guid CustomerId;
		public Email Email;		
	}
```
  Of course, these are a hypothetical solution in a hypothetical example. The relevant part is that you need to group the information based on a criteria. Here, I've chosen the "how often information is updated" criteria. Depending on the domain, the criteria may be obvious or you'll have to come up with one.

 Another thing to keep in mind is an event is a DTO so it should be kept as simple as possible, as serializable as possible. That's why you shouldn't put a full domain object in an event, but mainly primitives or at most value objects. The important thing here is the event has to contain all the relevant information that can be easily transported. Serializing rich objects is a waste if not a technical pain and keep in mind that a domain object is valid (make sense) only within a bounded context, while the event handler can belong to a very different context. Don't reuse the rich object  for event messaging, it's not [DRY](http://www.sapiensworks.com/blog/post/2014/01/15/DRY-Code-Rich-Code.aspx), it's an excuse for laziness, leading to complications in the future.

 A note about value objects. These are simple to serialize but might not have exactly the same meaning in all bounded contexts. In some situations it's better to just use the encapsulated values as primitives (string, int etc).

 Most of the time you'll have the combo: ActionCreated/ActionChanged. It's important to maintain the different semantics even if they are technically identical (they contain the same fields). Just make one inherit the other.

 There is a use case that might come up often: you want to notify that a status has changed. You have 2 options: make an event for each status or make an StatusChanged event containing the new status. In my experience, the second option is the more maintainable one. The downside is that an event handler has to check if it's the status it cares for. A small price to pay, in my opinion.

 If you have tips about domain event design please share your experience with a comment.


