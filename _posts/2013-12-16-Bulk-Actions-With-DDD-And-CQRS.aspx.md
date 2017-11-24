---
layout: post
title: Bulk Actions With DDD And CQRS
category: Domain Driven Design
---

One of my readers asked me

 
> "What about bulk actions? Say you've got your CQRS implementation and the UI has served up a grid with a 'Select all' and a button that enables the user to perform some action on all the select records. In your command processing would you simply iterate through the list of ids, re-hydrate the entities and do the relevant processing on each one?"
> 
>  
 Great question! It's a common scenario and one that usually makes you wonder "Remind me, why am I using DDD"? But let's start with something more concrete.

 Assume we have a list of messages, the user can select a number of them and press the 'Archive' button. The messages will be marked as 'Archived' and hidden from the normal view. First let's see the CRUD approach:

  
```csharp
update Messages set IsArchived=1 where Id in (@list)
```
  Short and swift with great performance.

 Now, let's see the DDD approach. Get all entities from the list of id, set the "Archived" property, send them to the Repository (which will save them in a transaction). Not so rocket science, but still more complicated and much slower than the CRUD approach (maybe there aren't 1000 messages but a number of 20 can be safely assumed). And if we use CQRS we have to update the read model too. DDD sure seems a more complicated way to do the same things as in CRUD.

 But what DDD allows you to have is flexibility. However you really need good domain modelling for that. The above approach is actually based on improper domain design i.e it's the approach for the _wrong_ modelling.

 Here's the trick: the "Archived" status of the message isn't part of the domain concept definition. Regardless of its state (Archived, Approved etc) a message is still a message (same with emails or e-commerce orders). The fact is we need that status in order to manage them and managing is an **use case** not a property or behaviour of the concept.

 So, let's leave the messages alone and use a MessagesManager (perhaps a Domain Service) to track the status of our messages. Or the number of likes, votes etc. The service interface looks like this

  
```csharp
public interface IManageMessages
{
    void Archive(params Guid[] id);
    //or
    void MarkAsRead(params Guid[] id);
    //or a more generic
    void SetStatus(MessageStatus status, params Guid[] id);
    IEnumerable<Guid> GetMessages(MessageStatus status);
}
```
  And the implementation can be as CRUDy as you like, because after all, it's just a simple table (Id,Status). And the update will be exactly the sql show above. One line, short and swift with maximum performance. The domain event might look like this

  
```csharp
public class MessageArchived
{
   public Guid[] MessageIds {get;set;}
}
```
  But let's stop a bit and think. All this message management is more of a read model context, rather than Domain's. How about we update the read model directly? The **MessageManager** could work with the read model so we don't touch the Domain model and we have maximum performance. The domain event is still generated even in this case.

 There's only one issue with this: if the read model is somehow corrupted, you don't have a 'source of truth' to rebuild the tracking functionality. That's why, depending on the app needs, perhaps it's safer to be a bit redundant. **MessageManager** should be a Domain Service (mostly write only but allowing you to get the messages state when rebuilding read models) which just stores the state and generates the domain event. Then, an event handler will actually update the read model.

 Yeah, still more complicated than the classic CRUD but the amount of additional code is quite low and you get A LOT of FLEXIBILITY and MAINTAINABILITY as you'll have each use case encapsulated in a service. Clean, decoupled, testable and scalable.

 Truth is, DDD (and CQRS - for me they work together) isn't just a trendy, over engineered way of doing things that apparently can be done faster with CRUD. It allows you to be agile, to be even CRUDy for some bits (but those are implementation details), to keep the code very clean and maintainable, with only a slight performance overhead compared to the simplistic CRUD approach. But... it's tricky to get right and it's so easy to fall into traps that will complicate the code a lot.

 Probably, the single advice I could give someone regarding DDD is: THINK a lot before you code. Then think again. If it seems complicated there's 99% chance you're doing it wrong. Think again and refactor.


