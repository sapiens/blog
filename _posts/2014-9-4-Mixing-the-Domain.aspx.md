---
layout: post
title: Mixing the Domain
category: Domain Driven Design
---

With DDD becoming more popular, it's like a "fight" between the old CRUD and the cool DDD. It's easy to assume that we have CRUD apps and DDD apps: this app is simple therefore it's CRUD, this app seems to have a rich domain, therefore we should be using DDD. But in practice, there are some apps which are 100% CRUD while no app is 100% DDD.

 Let's quickly define a CRUD app: it's a database UI concerned with taking the correct (valid) input from users to shove it into the db. It doesn't care about business semantics and contains NO business logic. It does contain some business rules represented by validation.

 But once you have 1 object/function encapsulating business logic, it's no longer a 100% CRUD app. But does it matter? A lot of devs are treating ANY app as a CRUD one (especially web apps) because they don't know better. And it works until the business logic becomes more complex and/or changes often. Then you have a maintainability problem.

 Noawadays, anyone (loose term) knows that for a domain rich in behaviour, DDD is the way to go. But regardless of how complex the Domain is, there's no such thing as 100% domain containing only Rich Behaviour Objects (RBO), you have at least one domain concept which really is a data structure (CustomerProfile comes in mind). Actually there are more than one, I say that a domain id at least 25% data structures.

 Thing is, with DDD we're usually using CQRS, Domain Events or Event Sourcing. But if a good chunk (actually one class is enough) of our domain is just a data structure, in other words it's CRUDy, aren't we complicating our life with at least 2 models (CQRS) or creating events which pretty much are the entity itself?

 The problem appears when we decide on a solution up front then force any problem into that solution. RBO with a CRUD mindset is pain, data structures with event Sourcing or CQRS is just a complication. How about we use the optimum solution for each problem? How about we're doing CRUD with the data structures from our Domain and CQRS, Domain Events and Event Sourcing with the RBOs?

 We don't need to declare an app as CRUD or DDD. It's an app, let's just use the approach that makes the most sense for each bit of the Domain. It's not about using the latest cool trend/design pattern, it's about coming up with the most maintainable solution. The goal is to solve the problem while making our lives easier. For big apps or where such flexibility is available, I'd say to also use the proper storage fit for the problem.

 This means that instead of "we're using Sql Server for everything.", you can use a doc db for your RBO, a stand-alone EventStore and a RDBMS in the same app. It's a hurtful mindset to think: "we do things only this (one) way (CRUD,SqlServer etc) because this works and this is how we've done it until now". It's bordering on stupid to choose the same solution regardless of the problem. An app is not a showcase for a technology or a design pattern, it's an implementation of a service used to bring value to its stakeholders.

 As a side note, this is why I'm implementing the persistence last. Even a small app can be 75% data structures and 25% RBO. I'll use CRUD for the data structures (that may involve an (micro)ORM) and CQRS for the reminder of 25%, with the domain objects serialized or stored as events for the objects that are fit for ES. But in order to know that, I have to design and implement the Domain first. Only after, I can start toying with the db. Anything before is wasted time and a poor design (as I'm biased to fit the Domain design to the db structures).

 In conclusion, your app is a place where you should use the solutions fit for the problem, all those design patterns can co-exist, the big mistake is to decide on a generic solution up front and then try to force all the problems into it.


