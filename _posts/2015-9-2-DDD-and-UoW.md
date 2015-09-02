---
layout: post
title: The Unit of Work and Transactions In Domain Driven Design
category: Domain driven design
---

As a principle, the Unit of Work (UoW) pattern is about ensuring consistency by **persisting** a group of changes as a unit (all at once). Basically, it's an all or nothing operation and the point of this is we don`t want to end up with inconsistencies. This is the main idea and it`s implemented by a db transaction or the ORM`s DbCOntext, ISession etc which themselves in the end will use a db transaction.

As you can see, the UoW is very useful and very persistence related. The problem is many devs trying to do DDD, end up needing something that surely looks like a UOW. However, its not.

Let's take a very simple example, the old 'I want to move an amount from one account to other' which from the Domain point of view is a domain transaction, which involves 2 instances of the same aggregate (the Account). It doesn't matter if the aggregates are different, what's important is that for the transaction to complete, we need to change 2 aggregates. This looks very similar to the Db's `BeginTransaction -> Commit` usage and naturally, devs consider this should be a UoW and try to implement it.

And that's how we end up with abominations like the UoW interface which 'holds' all the repositories that would be involved in the transaction. Too bad those devs are trying to implement a **domain use case** with a **persistence related design pattern**.

 Here's the thing, first of all, in a real world business process (a sequence of chained business use cases) there's very little of the 'all changes or nothing' mindset. Further more, a business process can involve different bounded contexts (BC) and can run for days until it  completes. Using the UoW pattern is a very naive solution and worse, from a technical point of view, it complicates things a lot when you're dealing with different db types or a distributed app.

Obviously, we need consistency in our process and at the end things will be consistent, but not immediately. As I've said above, a business process can be a series of use cases. Consistency is achieved when _all_ use cases involved in the process are completed. One at the time. Because, in the end, it's all about manipulating aggregates, and the only thing we need to care about is maintaining the involved aggregates consistency.

If you look at the real world, you'll see that very few things are immediate consistent, and 'thing' is the proper term, because **all real world processes** are _eventual consistent_, meaning they achieve consistency one operation at the time. Only whole objects are immediately consistent, because otherwise they wouldn't exist. Picture a chair (with 4 legs), if we remove a leg then it's not consistent anymore and it becomes a broken (not valid/usable) chair. The chair _needs immediate consistency_ for it to exist as a valid object.

In DDD, the concepts are implemented as aggregates,  therefore only aggregates need to be immediate consistent. But any _use case_ regardless of how many operations it has is basically a smaller process which is naturally eventual consistent.

 Btw, when dealing with a  business  process involving more than one BC i.e involves aggregates that are in different BCs, first of all, it means that we have at least one use case per bounded context. Each use case will publish domain events that will be used by the interested BC to 'trigger' their use cases. If at the end, we need to publish a notification that the process ended, then you're dealing with a saga i.e a long running process.

From a strategical DDD point of view, a domain use case encapsulates one **BC specific** business operation. This means that a domain use case is limited only to the model of the containing BC. If your use case needs to use an aggregate which is a part of another BC, then the modelling is wrong (either the use case or the BC).

So, a domain use case implementation is responsible to keep each involved aggregate consistent, but one at the time and this can be a problem from a technical point of view if the app goes down from whatever reason. That's why we need to ensure that a started use case/process needs to complete and we can use a durable service bus for that purpose (which will re-execute the use case). And we also need _every_ use case to be [idempotent](http://blog.sapiensworks.com/post/2015/08/26/How-To-Ensure-Idempotency) in order to prevent duplicate operations and this means that every contained aggregate change needs to be idempotent as well. The basic principle here is that we achieve idempotency by making every component idempotent.

In conclusion, a domain process or a domain use case should be considered as being eventual consistent and thus the UoW doesn't apply here. But aggregates need to be immediately consistent and thus, the UoW is usable as a repository implementation detail. But only for one aggregate (one consistency group) at the time.

P.S: The "transferring money from one account to another" example gives the impression that somehow this is a 'all or nothing' type of operation and we need to change both accounts in the same time to keep things consistent. Not really. It's still an eventual consistent operation (actually there are 2, one for each aggregate) and if the use case is interrupted midway because of a technical reason, it's not a problem, because the aggregates themselves are always in consistent state. What is 'inconsistent' is the use case itself, because it hasn't completed yet, but this is a non issue because a use case is technically a service, so it doesn't have its own persisted state therefore there's no consistency to care about. Other use cases depending on our case completion will wait for the 'TransferCompleted' event in order to start.
