---
layout: post
title: 7 Biggest Pitfalls When Doing Domain Driven Design
category: Domain Driven Design
---


### **Relational database mindset masked as DDD**

 This is so easily recognizable: it starts with data structures, finds one to many, many to many relations, loves the ORM and it knows that lazy loading is a necessary evil. Probably all devs doing this believe that DDD stands for Database Driven Design.

 Domain Driven Design it's not about the data or the relations between data. It's about modeling the relevant business concepts and processes. When modeling account and clients for a Bank, you would never hear the domain expert saying: "We have a many to many relations between accounts and clients". You might hear something like: "A client can open multiple accounts.Two clients can share the same account".

 Database mindset automatically 'knows' that a Client has many(a list of) Accounts. But the Domain mindset understands that a Client doesn't have an Account. The Bank opens up an account and gives the Client access to it. The Client never has an Account, he's merely an user of Account.

 Modeling Domain is a different task than modeling Persistence. Both are important but are different things and require a specific mindset. When modeling asks yourself what are you modeling: business processes or relational database schema.

 
### **Treating DDD as a recipe**

 DDD it isn't a way, a method of how to do things. It's a mindset to design an application. And it's a very simple mindset: the application **Design** is **Driven** by the business **Domain**. There are no steps of how to do DDD and there isn't a formula you can learn. What you can learn from Eric Evans or other DDD experts is how they approached a problem, but nobody can give you a sure recipe of how to do things.

 And one big mistake is to think that if guru X did it this way, clearly you should do the same. Every application is unique, what worked in one case might be a wrong and complicated approach for other case.

 
### **Starting with technical details**

 Begin modeling from a technical point of view (what are my aggregate roots, is Customer an entity, should I use an ORM, I have this Aggregate Root so I must have a Repository also etc) instead of the Domain's  i.e too much technical mumbo jumbo and too little 'philosophy'. Bounded contexts, aggregates, entities etc appear naturally when you understand the Domain. The point is: let the Domain guide the code and not viceversa. You don't try to fit the Domain into the programming language or a framework. You are just excodessing the Domain using code.

 I've seen a lot of debates if a method should belong to an Entity or to a Service. Is it a Domain Service, is it some other Service etc. These are just technical words which just complicate things. Just follow the Domain (the business) and see what it makes more sense from a business point of view, what it naturally fits. I repeat myself: there is no recipe and the technical aspect is secondary to the business aspect.

 
### **Being superficial about the Domain**

 It's easy to mistake the method for the purpose. In real life you put files into a folder. Why do you do it? Is it because you just want a place to put the files and it just happened to be a folder lying around or is it because you want to group the files according to a topic and the folder is the most obvious choice? What is the folder's main purpose: storage or grouping?

 
### **Treating the [Aggregate Root ](http://www.sapiensworks.com/blog/post/2012/04/18/DDD-Aggregates-And-Aggregates-Root-Explained.aspx)(AR) as a container**

 This is a very common side effect of the previous pitfall, it's very easy to consider that all an AR does is to 'hold' the Aggregate's other objects, especially Value Objects. This is tricky stuff actually, and the only way to not fall into the trap is to properly understand the domain.

 As an example, let's take the famous Order. Most of the time you'll find the Order as a collection of OrderLines which can be added or removed plus some methods to calculate totals, taxes etc. However.... an Order is not the Cart where you add/remove products until you submit the order. The Order is the list of products and quantities the customer ordered from you. That's it and nothing more and this means the Order is immutable. It doesn't even has methods to calculate totals because the client didn't order totals or taxes. All the order's financial data is in an Invoice. This means you don't add discounts to an Order either, again because the customer didn't order discounts.

 The Cart is a container of products and quantities until submitted when it becomes an Order which has an Invoice attached. The Order seems to 'hold' OrderLines but in fact, an Order can't exist without at least 1 OrderLine. A Cart can exists with 0 items in it, it's called an empty cart. Also, the importance of the Order is not that it's (in the end) just a list of OrderLines but a business document. The order lines are just an implementation detail.

 As a thumb rule, if all the AR does is to have a list of something, without representing a specific business concept, it's a strong hint you didn't model an AR but a simple container.

 
### **Tying the Domain to the ORM, an artifact of the database centric mindset**

 When using an ORM, the ORM becomes the relational database but without Sql and presented in an object oriented view. Mixing the ORM with the Domain it's like writing Sql from your business layer: a violation of the Separation of Concerns (SoC) principle. The ORM is not here for the Domain, it's here to protect developers from the ugly Sql.

 
### **The "One model to rule them all" approach**

  
  * [Bounded Context](http://www.sapiensworks.com/blog/post/2012/04/17/DDD-The-Bounded-Context-Explained.aspx) (BC).  
    A business concept may have slightly different definition depending on the context. In the Finance BC, a Product might mean only an id and a cost while in the Marketing BC, a product is an id, name and other relevant marketing details. The Inventory BC has all the technical details of the Product. The point is each BC understands something else when using the same concept. And the Inventory definition resemble but it's not the same as the Marketing definition. Even if they share the same name 'Product' they are different models, relevant only in their specific BC. 
  * Technical constraints.  
    Even if you have only one definition of the Product, if the object is not anemic i.e it is an aggregate root with behavior it will be a technical challenge to update and query the model. You can store it optimized for querying but it will be complicated to save/restore it from the db. You can serialize the object (trivial save/restore) but it's a pain (very slow) to query. In this case you need to have 2 models: one rich for updating and one 'dumb' for querying ([CQRS](http://www.sapiensworks.com/blog/post/2013/05/04/CQRS-Explained.aspx)). There's no need to complicate your life, good code is simple, clean code.  You know  you're applying DDD right when the Model is clear and flows naturally. Yes, it's a bit Zen like. And you can be pretty sure that you're doing it wrong when everything is so **complicated** that you cringe inside every time you have to change something. Remember this thumb rule: if it hurts then you're doing it wrong.


