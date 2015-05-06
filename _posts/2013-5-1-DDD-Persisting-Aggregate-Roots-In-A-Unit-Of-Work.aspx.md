---
layout: post
title: DDD - Persisting Aggregate Roots In A Unit Of Work
category: Domain Driven Design
---

There comes that time in DDD when you have to include two or more [Aggregate Roots](http://www.sapiensworks.com/blog/post/2012/04/18/DDD-Aggregates-And-Aggregates-Root-Explained.aspx) (AR) in a transaction (unit of work). They can be modified that's no problem, but how to persist them?! We have one repository per each AR and each AR actually defines the boundaries of its own transaction. How can we save all the Aggregate Roots in a Unit of Work?

 There are 2 cases:  
1. You are using one transactional storage like a RDBMS. In this case things are very simple: wrap the db transaction in a UnitOfWork and see that all involved repositories receive the db transaction via their constructor. This becomes even simpler if you're using an ORM.  
2. You are not using a transaction storage or there are multiple data stores involved. Even with a RDBMS, once there are 2 databases involved you can forget about transactions. What to do then?

 The good news is that you're using DDD so let's think a bit. What is the main advantage of DDD? You are modelling in code the business concepts and the business processes. This means that the business workflow can tell us how to achieve that transaction.

 Let's consider this example: we have an Inventory and an Order. These are our AR and we want to simply move the goods from the inventory to the order, which in code is just something like subtract quantity from Inventory and add the same quantity to Order. A typical transaction.

 But let's see how it would be done in the real world. Let's say that we have 2 rooms: Inventory and Order. First of all, you don't just add/take goods from an Inventory without a business reason. Stock is increased when the suppliers delivers the goods and decreases when the goods are moved to the Order (or other business activity). Same with Order, goods don't just appear from thin air. Usually they come from the Inventory. This aspects are important in understanding how we should model the transaction.

 So, we want to move 3 goods from Inventory to Order. This means that you take the 3 items and transport them to the Order room. In this period of time, the Inventory will have (stock-3) items, while the Order is unchanged. But this is not a problem, it's something normal, you can't just teleport things around and even if you could, there will still be some slight delay. Until the items are put in the Order room, we can say that there is some inconsistency with our inventory but that doesn't bother anyone because things will become consistent quickly.

 The goods arrive in the Order room so the transaction is successful. But what if there is a problem? If the Inventory doesn't have enough goods, then nothing will be sent to the Order and the transaction is refused on the spot. But what if the Order refuses the goods (because it already has them)? In that case, the natural way is to return the goods to the inventory. Once returned, the Inventory will have the same stock as before while the Order doesn't change anyway. It's a real life **rollback**.

 Knowing these things, it becomes trivial to implement them in code. But here is a tiny catch: it requires a message driven approach and being very comfortable with the concept of eventual consistency.

 So, we have Inventory and Order aggregate roots used in a unit of work. Instead of doing: begin transaction -> modify inventory -> modify order -> save inventory -> save order -> commit , now we'll use the workflow as it happens in the real business.

 You don't say to Inventory: 'substract 3 from stock" you say: "Inventory **AllocateQuantityForOrder**". If it can do it, the Inventory modifies its state then generates an event: **GoodsAllocatedForOrder**. One of the handlers of that event will update the Order: **ReceiveGoodsFromInventory**. If all it's ok, the Order change its state. If not, the event handler will issue a **ReturnGoodsToInventory** command, which is basically a rollback for the Inventory.

 And here you have it, a transaction involving 2 aggregate roots that doesn't depend on a transactional storage. Let's see the differences between these 2 solutions.

 First option is the conventional one, the simplest to understand and to use but:  
- it requires at max one transactional storage.  
- it requires that the modifications be made in the same time, you can't span the transaction over 2 web requests for example.  
- you can't use more than one storage.  
- poor scalability.  
- it doesn't leverage DDD.

 Second option is clearly a bit more exotic (if you're not used to message driven apps) but:  
- it takes advantage of DDD thus it's modeled by the business process, so it's a natural fit.  
- works in a distributed environment.  
- it's storage agnostic.  
- it can be scaled easily.  
- it requires good developers and a proper understanding of the Domain and being comfortable with message driven architecture.

 I favor the second approach, but the first option can be the pragmatical solution for some cases. It's important to take your time to think and choose what option fits your app better. Remember that the devil hides in the details so don't be hasty about your decision.


