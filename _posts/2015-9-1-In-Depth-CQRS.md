---
layout: post
title: An In-Depth Look At CQRS
category: CQRS
---

While in theory CQRS simply means that the model for reads(query) is different (a simpler version of) than the model for writes(command), in practice things are a bit more complex. Add Event Sourcing and CQS into equation and things become quite confusing. But first things first.

## Know Thy Enemy

You shouldn't use it just because you've heard it's cool and this how apps should be developed. You need to be aware of its benefits (and its drawbacks). The main benefit is that it enables an app to perform very fast reads while having a maintainable domain model. The drawbacks are that it requires a bit of experience in order to be implemented properly and it adds more technical complexity. Maintaining 2 models (and the read model can be further specialized into more models) adds more code and might require a certain infrastructure. That's why it might be overkill for a glorified db UI app, like many CRUD apps are.

## CQRS is a different concept than CQS

CQRS not only has a very similar name to CQS but it's actually inspired by the latter. But what's CQS?

CQS (Command Query Separation) is a design principle which states that an operation should do either a Write (Command) or a Read(Query) but not both in the same time (we`re talking about high level here, not a low level implementation which can do both read and writes). At first it was a principle used when defining functions, but it can be used as an architectural principle too: the application is "partitioned" into **Commands** (services that change something) and **Queries** (services which only read the model). It's great for maintainability, because each command/query encapsulates one use case.

If we're using a message driven architecture then we're gonna have commands and command handlers and queries and query handlers. Obviously, in a DDD app,there will be events and event handlers too, but those are have nothing to do with CQS. Note that a command/query handler is just an implementation detail of the Command/Query part of CQS. Basically, I can decide that all my command services will be implemented as command handlers, in order to take advantage of a service bus. Again, this is an implementation detail.

These have little do to with CQRS which deals with a Command **model** and a Query **model**. To clear up the confusion you can consider than CQRS is about separate models while CQS is about separate behaviour **responsibility** (although these can overlap).

To make things even more fun, CQRS and CQS work _very well_ together, however they're still different concepts and should be treated as such. So, an app can use CQS without touching CQRS and vice-versa. However, in practice, especially in DDD apps, they're often used together in combination with a message driven architecture.

## CQRS Can Be Used With Event Sourcing (ES)

...but one doesn't always imply the other. And speaking about events, an event driven architecture or the Domain Events pattern doesn't imply ES. They're all different things that are used together because they have great synergy. In probably 99.99% of cases, an app using ES will also use CQRS (and a message driven architecture), because it needs a queryable read model, different than the even store model (hence CQRS).

## CQRS: Command Model

Well, "command model" is not a very accurate term, because in practice there are 2 models: application(as in non-persistence) and persistence. For now on, when I'm referring to the application, I'll say "business or domain" because this is where things make more sense. But it's good to know that we're talking about principles that work and can be used outside the domain layer (if you're thinking in layers).

So, we have 2 points of view of a command model: the model which encapsulates business rules and the model used to store the aggregates (business objects). The command application model is always about enforcing business rules. This is the 'behaviour rich' domain model. It's used only for changes and it defines the "domain model".

But we do need to persist/restore this model so we need a command _persistence_ model fit for this purpose. And you can store it in way you like, however the easiest ways is to use an event store (if your using ES which I highly recommend for DDD apps) or to just serialize the entities or use a document database. Obviously, you can use an ORM, but this is the complicated solution, you'll need to define mappings for each entity and tell it how to deals with Value Objects and other encapsulated details that might be required.

As I've said, it's complicated and it's the least maintainable solution. A generic storage like the event store or a document db, basically anything that doesn't care about your entity structure is the desired solution. And once you have that storage it will work with any entity automatically and in a way it's a kind of a generic repository, but the good one, not the anti-pattern abstraction on top of the ORM one.

We can get away with a generic, non-queryable store because we only need basic CRUD functionality from it: Add, Get, Save, Delete. For the use case which require more than one entity, the the service will use a domain query which will use both the query persistence model and command persistence model to do its job.


## CQRS: Query Model

As with the command model, we still have the query application model and the query persistence model. I'll start with persistence because it's the easiest one: this is your queryable model plain and simple. With a RDBMS this is where you're writing SQL, doing joins and other fun stuff. With a document db you might have query indexes. You can also have very specific models i.e serialized pre-computed view models or query application models. Pretty much anything that will allow you to query things fast.

The application model has the following 'feature': it doesn't contain/enforce _any_ business rules. It can use encapsulation and can be immutable and even have some _technical_ behaviour (methods), but nothing domain related. In many cases it's just a data structure or a DTO (Data Transfer Object). If we're using CQS, then the query application model is in fact the query handler and its result.

Note that a domain (which is the Command model) has queries. However in this case, it's the CQS principle at work and the query service can use a query persistence model, but it can return business objects. This is a matter of using CQS 'inside' the Command of CQRS. It is a bit confusing but if you understand that these principles are different yet they work together, then you won't have a problem.


## Context

It's always important to understand the context we're working on. You might call these layers or components, vertical slices etc. but the important part is that they're a criteria to group related concepts and use cases. And we can mix CQRS and CQS. So we can have an UI Query model which consists of query handlers querying a persistence read model and returning view models (whole or bits of it). An Application context with Commands (Register Account, Login etc) and Queries (GetUserForLogin, GetUserRights etc) or a Domain context which has their very own Command Model and/or Command Handlers as well as Queries (CountVipMembers, GetMembersWithoutOrders etc).

As a fun fact, once you see the app like a group of contexts which themselves are a group of use cases, the traditional 'layer' approach doesn't make much sense anymore. It's all about specialized model for changes or for reads and clean services which either change something or read something. And these lead to a better (more maintainable and scalable) architecture but that's a subject for a future post.

One more thing, there are no rules with CQRS besides the separation of the write/read concerns. What matters the most is the principle and the problem it solves. In some use cases, applying the principle means there's no clear separation between the writes and the reads from one point of view (you might be using only one persistence model). This doesn't invalidate the whole principle, it's just a local decision to implement things differently. As long as the app is maintainable, there is no problem. Like many other principles, CQRS is not a recipe with a strict "how to" dogma. But everyone using CQRS should first understand it properly before deciding on a specific  implementation.
