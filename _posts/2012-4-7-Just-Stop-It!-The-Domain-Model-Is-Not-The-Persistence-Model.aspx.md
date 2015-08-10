---
layout: post
title: Just Stop It! The Domain Model Is Not The Persistence Model
category: Domain Driven Design
---

I think 90% of times I see a DDD question on StackOverflow, it's something like this: 'I have this domain object' and the code shows an EF (Entity Framework) or NH (NHibernate) entity.

 **NO!** This is WRONG (99% of times anyway)!

 I've said it [before](http://www.sapiensworks.com/blog/post/2012/02/08/Understanding-the-Model-in-MVC.aspx) and I repeat it:


  * The **Domain Model** models real-life problems and solutions, it models BEHAVIOR.
  * The **Persistence Model** models what and how data is stored, it models STORAGE STRUCTURE.  See? They have pretty different  purposes. The domain is the reason the application exists and everything gravitates around it. The domain should not depend on anything,especially not on a persistence IMPLEMENTATION DETAIL like EF or NH. When you design the Domain Entities, they don't know anything about persistence. Persistence, database, doesn't exist.

 When you design the Persistence Layer, that layer serves the Domain and depends on it. So the persistence needs to know about the domain but not vice-versa. You design the persistence entities for storage purposes and to match the ORM's constraints (like making all the properties virtual). So you'll have Domain Entities and Persistence Entities, each with their own different purposes and implementations.

 Yes, they do resemble and sometimes can be identical (when the domain is very simple) but that's nothing more than a mere coincidence. Every time you're modeling something in a repository or using an ORM , you are modelling the persistence NOT the domain. There is a reason Eric Evans recommends to start the app with the domain and to IGNORE anything db related: the Domain should not be tainted with infrastructure details, because most of the people start everything with a database centric approach. And once you start with the db, everything will evolve around it and will be constrained by it. But you don't build the application for the database, you build it for the Domain, the database is just a Persistence implementation detail.

 In conclusion, repeat after me: the Domain Model is NOT the same as the Persistence Model. Each serve a different layer with different responsibilities.
