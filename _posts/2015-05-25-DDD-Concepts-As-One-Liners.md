---
layout: post
title: DDD Concepts As One Liners
category: Domain Driven Design
---

**Bounded Context** : all models sharing the same domain language

**Aggregate**:

- DDD's command model representing a domain concept or a relationship between domain concepts
- a consistency boundary around a domain concept, often represented as an object graph (from [Mathias](https://github.com/mathiasverraes))


**Aggregate Root**: a part of the Aggregate in charge of enforcing the aggregate's consistency and acts as its 'representative'

**Entity**: a business object representing a domain concept with a stable identity throughout changes over time

**Value Object**: an immutable object representing a (composite) value that has domain significance

**Domain Service**: encapsulation of pure domain behaviour that doesn't have its own persisted state e.g calculate tax

**Application Service**: acts as an adapter between the app's clients and the domain 

**Domain Event**: a data structure representing one domain change

**Repository** : the object responsible for persisting/loading aggregates

**Criteria**: it tells the Repository _what_ kind of aggregates to retrieve

**Specification**: an implementation of a business rule aspect

**Eventual Consistency**: a process involving multiple aggregates modifies (and persists) one aggregate at the time, but not in an atomic fashion

**Unit of Work**: basically, an [anti pattern](http://blog.sapiensworks.com/post/2014/06/04/Unit-Of-Work-is-the-new-Singleton.aspx) 

**CQRS**: have one model (behaviour and data) handling business state changes and at least one _other_ model handling query use cases

**Event Sourcing**: express business state as a stream of events  

 
 
Want to contribute? [Send me a PR](https://github.com/sapiens/blog/blob/gh-pages/_posts/2015-05-25-DDD-Concepts-As-One-Liners.md) 
      
