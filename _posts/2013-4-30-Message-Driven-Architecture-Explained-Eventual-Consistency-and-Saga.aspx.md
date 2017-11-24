---
layout: post
title: Message Driven Architecture Explained - Eventual Consistency and Saga
category: Architecture
---

You can find [part one here](http://www.sapiensworks.com/blog/post/2013/04/19/Message-Driven-Architecture-Explained-Basics.aspx)

 Let's talk about the really interesting stuff about Message Driven Architecture (MDA).

 
## Eventual Consistency

 Most of the time you might use a RDBMS which enforces consistency and you know that if the transaction has been committed, everything is saved. When dealing with distributed components availble on different servers, suddenly you can't use this feature. The same problem appears in a local application when you're using NoSql db or you have to save data in 2 databases belonging to a RDBMS or you have 2 or more incompatible data storage (for example: file system and rdbms).

 With MDA you need get used to **eventual consistency** which can be explained very simply: everything will be consistent.. eventually. The thing is, a model is consistent at one moment in time. For example, a web page shows some data. The data is a valid model until you refresh the page. If you don't refresh it then for you the model doesn't change, even if on server it's already different. However, eventually, you will refresh the page and you'll get the latest model which of course might change 1ms after the page loaded.

 Eventual consistency is everywhere around us, we all work with obsolete data, but the RDBMS have spoiled us to think that we can have always access to the latest, accurate and 'true' data. This works when using only one RDBMS but that's it. Change that condition and we're back to the real world where everything gets stale after a second. Embracing (the natural) eventuality is required for MDA and it's not that hard to get used to it, actually.

 So, how do we deal with it? Well, quite simple actually. Since in MDA every business object is modified as the result of a command, it means that the commands themselves need to be committed together as a unit of work. Every ServiceBus has the feature to send more messages at once. This is the 'transaction'. Note that the important part is that the commands will be received together, but that doesn't mean they will be handled at the same time. But this doesn't matter as they will be all handled eventually :)

 
## Saga

 A Saga is used to manage a long running process. Got it? No? Well, there are some long running operations which are considered to be complete when all their steps are completed. Nothing unusual except the fact that each step can be completed independently of other so there isn't a predefined order. Another aspect is that steps are completed in unknown time periods, so one can take 2ms while others minutes, hours or days. The saga keeps track of these steps as they happen and ends when all involved steps are completed.

 Let's consider this real-life example: John goes to a fast food restaurant and orders a menu. This means one hamburger, one portion of fries and one soda. John tries to pays with the credit card... if he finds it. For whatever reason, it's not in the usual place. In the mean time, the employee get the soda and tells John he has to wait a bit for the hamburgers and the fries. John finally finds the credit card and pays. After a few minutes everything is ready and he can enjoy his meal.

 Now, in this example we don't care much about John (sorry!) but we care about the operation as seen from the restaurant point of view. The operation is complete when the whole menu is ready and paid for by the customer. We have the following steps:  
- Hamburger ready;  
- Fries ready;  
- Soda ready;  
- Payment received.

 The important thing here is that any of these can be completed in any order. Remember in our example that the order was:  
- Soda ready  
- Payment received  
- Hamburger ready  
- Fries ready

 Usually things probably happen in a slightly different order depending on how you pay and if the menu items are already prepared.

 Now, let's 'write' the above example as a saga.

 John orders a menu, that is the OrderSubmitted event is published which starts the operation (saga). Starting the saga means issuing the commands: PrepareHamburger, PrepareFries, PrepareSoda, ReceivePayment. It just happens that E will handle all the commands: first E checks for each menu item to see if they are available. Hamburgers and Fries are not so two other commands MakeHamburger and MakeFries are sent. The Soda is available, so the SodaReady event can be published which signals the saga that one step has been completed.

 In the mean time John pays so the PaymentReceived event is published and saga marks another step as completed. After a few minutes the HamburgerReady and FriesReady events arrive and the saga marks the last items' completition. All the steps are completed so saga ends, preferably with the SentOrderToClient command.

 Here we have 2 actors: the employee (which is the messages handler) and the saga. A saga is started by an event and it's updated by other events. All these need to be correlated so there is usually a message property which acts as the correlation id (for example OrderId). All that saga does is to coordinate events and commands. In the above example, saga began by sending a few commands then handled the events which updated its progress and when it completed, it sent a final command.

 In this example 'long running'  meant a few minutes. But consider the case where you order something online, you pay now but the seller may get the payment confirmation after 5 minutes, the items will be available in 3 days and you'll get the products delivered after 1 more day. The PrepareForDeliverySaga starts with OrderSubmitted and ends when both PaymentReceived and ProductsInStock events are published. The saga may end with a ShipProducts command which will generate a OrderShipped event which in turn may start a ShippingSaga which ends when the customer has received and signed for the goods. Or you can include the OrderShipped and OrderDelivered steps in the first saga.

 And this should be all, I hope now you have a better understanding of what Eventual Consistency and Saga are.


