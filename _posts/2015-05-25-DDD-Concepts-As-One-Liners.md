---
layout: post
title: DDD Concepts As One Liners
category: Domain Driven Design
---

**Bounded Context** : all models sharing the same domain language

**Aggregate**:
- a group of objects that need to be consistent together in order to represent a valid domain concept
- a consistency boundary around a domain concept, often represented as an object graph (from [Mathias](https://github.com/mathiasverraes))


**Aggregate Root**: the object from the Aggregate that enforces the aggregate's consistency and acts as its 'representative'

**Entity**: a business object representing a domain concept with a stable identity throughout changes over time

**Value Object**: an immutable object representing a (composite) value that has domain significance

**Domain Service**: implements a domain use case and/or provides domain utilities

**Application Service**: acts as an adapter between the app's clients and the domain 

**Domain Event**: a data structure representing one domain change

**Repository** : the object responsible for persisting/loading aggregates

**Criteria**: it tells the Repository _what_ kind of aggregates to retrieve

**Specification**: an implemention of a business rule aspect

**Eventual Consistency**: an use case involving multiple aggregates modifies (and persists) one aggregate at the time, but not in an atomic fashion

**Unit of Work**: basically, an [anti pattern](http://blog.sapiensworks.com/post/2014/06/04/Unit-Of-Work-is-the-new-Singleton.aspx) 

**CQRS**: have one model (behaviour and data) handling business state changes and at least one _other_ model handling non-domain queries

**Event Sourcing**: express a business object state as a stream of events  

 
 
Want to contribute? [Send me a PR](https://github.com/sapiens/blog/blob/gh-pages/_posts/2015-05-25-DDD-Concepts-As-One-Liners.md) 
      
