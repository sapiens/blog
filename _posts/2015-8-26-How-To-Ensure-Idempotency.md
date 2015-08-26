---
layout: post
title: How To Ensure Idempotency In An Eventual Consistent DDD/CQRS Application
category: Domain driven design
---

If you want a _reliable_  DDD/CQRS app (distributed or not) you **need** to treat idempotency as a first class concept. And while not all DDD apps use a message driven architecture, most of them do so and, in my opinion, that's the best way to end up with a maintainable codebase. Idempotency is required because of the _eventual consistency_ 'feature' of a message driven system.

But before we talk about solutions, let's understand the problem. An eventual consistent app treats a business process as a sequence of business cases. An aggregate is changed, one or more events are published, the events handlers trigger other business cases and so on. We have a chain of independent operations each with their own state which together define one process (a domain unit of work), that can be broken at any time, because the app crashes or the server recycles the app or we deploy a newer version etc. We end up with a state where some aggregates were changed, while others never made it.

It sure looks like the app is in an inconsistent state, however, the use of the durable service bus will make sure that we are in an _eventually consistent_ state i.e things will be consistent very soon (it's just a matter of time). If you're used to the lower level db transaction, where everything is either commited or rolledback, eventual consistency looks quite dangerous and you might feel unsafe. Luckily, it's just the typical "out of the comfort zone" feeling. Eventual consistency might look fragile and complicated to handle but once you get used to it, it will become second nature.

The main 'ingredient' in dealing with eventual consistency is proper idempotency handling. Considering a process can be interrupted at any time and then executed again from the beginning, we need a way to ensure that a repeated operation changes a model only once. I'm going to show you 2 ways of doing this, but first there are some "rules":
- Every operation has an unique id (guid);
- The aggregate root  needs the operation id to be set before any changes.

## Handling Idempotency at the Aggregate Level

The easiest scenario is when you're using Event Sourcing. And that's because the entity already has all or at least the latest events. And we can make the operation id part of the event itself. All my events(and commands) have the `SourceId` property which is the id of the operation that generated the events (by default it's the command id which for commands, in most cases is also the source id). Then it goes like this

```csharp
var foo=_repo.Get(fooId);
IEnumerable<IEvent> evnts=new IEvent[0];

try
{
  foo.SetOperationId(cmd.SourceId);
  foo.ChangeStuff();
  evnts=foo.GetGeneratedEvents();
}
catch(DuplicateOperationException ex)
{
  evnts=foo.GetEventsForOperation(cmd.Id);
}

_bus.Publish(evnts);

```

So, the aggregate is aware of the most recent (or all) operation Id and if you try to set one that was used before, it throws. Pretty simple.

But what happens if you're not using event sourcing? Well, we can make the entity keep a history of operations and check it everytime the operation id is set. The problem with that is it will increase the entity size a lot if the object is changed often. However, for objects that rarely change it might be a solution. And here's a base class that may be used for that.


```csharp
public abstract class AIdempotentEntity:IOperationId,IEntityId
  {
      protected int MaxHistory = 2;

      private List<Guid> _history = new List<Guid>(10);
      protected Guid? _operationId;

     // json serialization friendly
      protected List<Guid> History
      {
          get { return _history; }
          set { _history = value; }
      }

      public virtual void SetOperationId(Guid id)
      {
          if (_history.Contains(id)) throw new DuplicateOperationException(id, null);
          _operationId = id;
          if (MaxHistory == 0) return;
          var pos = _history.Count;
          if (pos == MaxHistory)
          {
              pos = 0;
              _history.RemoveAt(MaxHistory-1);
          }

          _history.Insert(pos, id);


      }

      public Guid GetOperationId()
      {
          return _operationId.Value;
      }


      public Guid EntityId { get; protected set; }
  }

```

So, what can we do when you have objects that change often or you need to keep all the operation id history? We'll handle idempotency at the persistence level.

## Handling Idempotency at the Persistence Level

I must say this is my preferred solution. It's also a generic solution that can be used any time you need to ensure idempotency but it does have a small flaw: it requires an ACID compliant db.

The main idea is like this: we have a table (OperationId:Guid,Hash:string) with a unique constraint on both columns. When we persist an object, we have this transaction

```csharp
using (var t = db.BeginTransaction())
 {
     if (db.IsDuplicateOperation(item.ToIdempotencyId()))
     {
         return;
     }

    //save object

    t.Commit();
  }


//ext method
public static bool IsDuplicateOperation(this DbConnection db, IdempotencyId data)
       {
           data.MustNotBeNull();
           try
           {
               //idems is the name of the idempotency table
               db.Insert("Idems", data);
           }
           catch (DbException ex)
           {
              //duplicate operation
               if (ex.IsUniqueViolation()) return true;
               throw;
           }
           return false;
       }

   public class IdempotencyId
      {
          public static IdempotencyId Empty=new IdempotencyId() ;
          public bool IsEmpty() => OperationId == Guid.Empty && Hash.IsNullOrEmpty();

          public Guid OperationId { get; set; }
          /// <summary>
          /// Its actually an identifier of the entity/model/event involved
          /// </summary>
          public string Hash { get; set; }

          public IdempotencyId(Guid operationId,string hash)
          {
              OperationId = operationId;
              Hash = hash;
          }

          private IdempotencyId()
          {

          }
      }

```

Basically, we try to insert an idempotency id that uniquely identifies the object and the operation and if it fails, it means it's a duplicate operation. If it succeeds then we can persist the entity and commit the transaction. If an error occurs the transaction is rolledback and the idempotency id is not saved.

The idempotency id is the combination of operation id and some other specific identifier(I call it a hash in lack of a better name) of the object. For domain objects, the hash can be the entity id. For read models, hash can be the read model category and/or the domain entity id involved. Obviously, it depends, there are no rules here, the only 'rule' is that you need to be aware that one operation can affect multiple aggregates or read models. For example, if you compute metrics in an event handler and this involves incrementing a counter, the read model specific hash should be something like "stats_user_count".

Note that for this to work we always need to have a transaction involving the idempotency storage and the domain storage, so they both need to share the same storage. But you don't need one global idempotency store, each component can have its very own (also the case if you're using multiple databases/engines).

## Conclusion

There you have it, 2 approaches to ensure idempotency for your eventual consistent app. They both work and I'm using both at the same time, it's not about choosing one over the other upfront, but choosing the best for your specific case. As an example, I have a "Project" entity uses event sourcing and it makes sense to use the first approach, but the read model updater (an event handler) and the metrics updater (another event handler) subscribing to the ProjectCreated event will use the second approach with the idempotency id hashes: 'project_read' and 'project_count'.
