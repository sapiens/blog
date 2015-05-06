---
layout: post
title: CQRS Explained
category: Architecture
---

I find it refreshing that CQRS is more and more used. What surprises me is the for many developers this is considered a 'big leap', something involving learning 'advanced' stuff and complex frameworks. But CQRS is just a very simple and straightforward concept: Command Query Responsibility Segregation (Separation).

 Let's take it step by step.

 A **Command** is behavior which modifies the Model. A **Query** is well... querying (reads) the Model. Why the separation? Think a bit: modifying the Model usually means business rules validation and interactions between business objects. It's a 'rich' Model. When we want to query the same model, we have to restore it from persistence (which can be very slow), populate all the objects, instantiating business rules etc. All this just to get the Name for some Product. How about the names for 1500 Products?

 Sure we can store things directly in a queryable form, that's why you're using a RDBMS, but restoring the state especially when dealing with complex relations between the objects it's a complicated affair.

 But someone smart pondered the things and wondered: querying the model doesn't change it, so instead of complicating my life, how about I use models optimized for my tasks? If I want to change the model, I'll have a rich and relevant business model but when I want to query it, I'll have a simplified, optimized for reads model. So, instead of having only one Model trying to please two different scenarios, I'll have one model optimized for each scenario.

 But wait, doesn't that mean we have to write more code and breaking the DRY (Don't Repeat Yourself) principle? Not quite. First of all the query model, usually referred to as the read model is just a slimmed down version of the richer model. Of course, depending on the contexts you could have more than 1 read model.

 The DRY principle isn't violated, because our read model is just 'dumb' data. The advantage is that each context gets its own relevant and most importantly **easy to use** model which simplifies the understanding and the maintenance of the code.

 And this is all that CQRS means: have a different model for changing state (command) and for reading state (query). That's it, nothing more and nothing less

 I'm sure that searching about CQRS you came across CQRS frameworks and Event Sourcing, but I'm afraid that CQRS only means the simple thing stated above. You don't need a framework for such concept, it's impossible to have a framework. What frameworks/toolkits you've found are in fact for Domain Driven Design or Event Sourcing (ES) or Domain Events etc which happen to use CQRS.

 CQRS is a great concept to be used in any application when one model means complicated code and that's why is a perfect fit with DDD, ES or a Message Driven Architecture. You'll find them used together, but each means something else and a good developer should understand what each pattern brings to the table. They are tools not buzzwords.


