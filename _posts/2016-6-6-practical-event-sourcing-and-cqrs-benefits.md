---
layout: post
title: Practical Event Sourcing And CQRS Benefits
category: Domain Driven Design
---

If you're reading this post, you already know Event Sourcing (ES), but I want to look at things from a higher level point of view than usual. 

## We have a more strategic, domain driven approach
We know ES is about Domain Events. But a Domain Event is just a technical a way of saying **business state change**. Instead of being concerned with objects, data structures or db schemas, we're modelling things closer to how the business functionality is. We can't do proper ES if you don't _identify_ the relevant (from the business point of view) domain state changes, **the events** (sic) that are shaping the domain state and activity. Sure, you can design your Domain Events (you shouldn't!) and use the most technically correct ES with a nice Event Store and CQRS and messaging etc, but you'd be throwing away the strategic advantage which allows you to build maintainable apps. It's not about learning and applying a recipe, but adopting a certain mindset.     

While the recipe part (implementation) is simple to learn, the first main benefit of ES is that we look at the domain as an always evolving event-driven system. 

## Business state is just a snapshot of a business process
And I'm not talking about EventStore snapshots, that's an implementation detail. When the domain state is **expressed** as a stream of events, we're just _recording_ the relevant bits of the business process. And we can navigate through the 'timeline' from the beginning til the current moment, seeing what the business state is at various points in time. Basically, we have the role of business activity custodians, in charge of enforcing the correct history. It's a little bit more than just storing data structures in a database, isn't it? ;) 

## One source of "truth", but multiple views
And speaking of correctness, the recorded events are the only stored information that we can trust as being valid and consistent. This means that once an event has been recorded (stored), it shouldn't be changed. It's part of the history now, it's **immutable**, set in stone, read-only etc. We don't rewrite history here! 

We can't change the past but we can look at it from different points of view so, we can project that stream of events into specialized read-only models, containing only the relevant data for a certain use case. And we can store those models as a technical convenience. 

## Working with persistence is easier
The events don't appear from thin air. They are the outcome of a business use case or process. When we want to change things, we're using a model specifically designed for that purpose. In CQRS it's called a _command model_, in DDD is called an _aggregate_. That model contains the relevant business rules, and based on them and the user input it outputs one or more changes, **expressed** as domain events.

The beautiful thing here is that, regardless of how simple or complex a model is, the result is always a **bunch of events** that are sent to an Event Store. No mapping from objects to tables, no db schema required, no serialization gotchas. Restoring a previous state simply means replaying the recorded events. No memento, manual or automatic mapping involved. Best of all, you don't need to expose anything that a business object doesn't need to expose. In ES a business object rarely has public properties.

Sure, you might say "it's simple when we deal with the command model, but what about queries? Maintaining a separate read model is added complexity". Hm... yes and no.
First of all, a read model can refer to persistence schema and/or an object that represents a read-only view of some concept. In ES we need a queryable persistence model, however that's like having a simple CRUD app where the data has already been validated and where constraints don't really matter. It literally is a matter of updating/querying some tables, things a junior can do easily. 2 simple tasks are always performed faster than 1 complex task.

## Easier to unit test
Initializing a model's state to test a certain behaviour is trivial. Just create the events and feed them to the model. Testing a command model is trivial; invoke the command method then just ask the model for the generated events. 'nuff said.

## Increased developer productivity and a maintainable and scalable application
Not messing around with ORMs, or 'adjusting' objects to fit the persistence tool, means less complications and more value delivered. Properly understanding the domain, (you know, the functionality we need to implement) means we're focusing on the relevant things, writing less code and/or simpler solutions. 

Our models are more granular, therefore easier to understand and to maintain (time saved). We understand and embrace the different nature of changing the business state and reading state and we optimize for each scenario. Recording events in their raw form, instead of storing a limited interpretation, allows us the flexibility of multiple views and the ability to deliver future value. Best of all, the 'hard' part of all this is to pay attention to the domain expert, the technical implementation is simple enough.

   


  