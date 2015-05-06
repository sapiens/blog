---
layout: post
title: DDD: Identifying Bounded Contexts and Aggregates, Entities and Value Objects
category: Domain Driven Design
---

Let me be clear about one thing concerning Domain objects: they aren't either Entities or Value Objects (VO). You can have simple objects in your Domain **and** you can have objects which have a business meaning. Only an object _recodesenting a Domain concept_ can be classified as an Entity (it has an id) or a VO (it encapsulates a simple or composite value).

 
## Bounded Context (BC)

 First thing before starting an app is to identify the Bounded Contexts. Luckily this is quite simple. First of all let's remember that a BC simply defines a model that is valid (makes sense) only in that BC. If you're designing your app as a group of autonomous components (and you should) aka vertical slices aka microservices (for distributed apps) working together, then each component is a BC. For example, I'm working on a Saas app and these are some of the components: Membership, Subscription, Invoicing, Api Server, Api Processor. Each is like a mini app in itself, each has their own domain and thus their own model which makes sense only for that component (BC).

 The components being autonomous, they don't depend one on another (at most they know about other's abstractions) so they don't share the model i.e the model of one BC doesn't cross its boundaries to be available in other BC. But, a subscription is related to an account so it might seem that **Subscription** must know about **Membership** model. Similarly, Invoicing is related to an account and needs data usually found in **Subscription**. How do we solve this, without coupling the components (breaking the boundaries)?

 Each BC has its very own definitions of concepts even id it happens that they share the same name. An **Account** is a fully defined concept in **Membership** but it's just an id in **Subscription**, **Invoicing** or any other BC and that's because the BC itself controls the definition. So the Account is not just an id because the Membership Account entity has a property Id, but because the other component said so. However, in practice, a concept which is an entity in other BC will at least be defined as an id in other BCs. But the VO are a different story and each BC really has their own definitions. Of course, some concepts may be generic enough that their definitions will be identical. But that's just a coincidence, and in fairness, if a BC shares a lot of concept and definitions with another, perhaps they should be merged.

 What about the data that a BC might need from another? A BC should be like a black box. You don't see what's inside, you know only about input and output. In a message driven app, this means you send commands to be handled and you get events. Those are DTOs and contain all the relevant (in/out)data for an use case. So we have our BC listening to other BC's events. If you really want to keep things decoupled, because you want a reusable component, then you can use a mediator that will listen to one BC then send the relevant data to the other one. That's the approach I've taken with the **Invoicing** component (so that it wouldn't know about anything **Subscription**).

 So, a BC takes DTOs or commands as input then publishes events as output. A BC model exists only in that BC and all the DTOs are just flatten aspects of that model. Since an event is just a data structure with no business rules attached (consider it a read model ;) ) that data can travel to any other BC that can extract the relevant information from it. In the end, BCs 'communicate' via simple data that only have the role of input/output.

 
## Aggregates

 Identifying **Aggregates** is codetty hard. An aggregate is a group of objects that must be **consistent together**. But you can't just pick some objects and say: this is an aggregate. You start with modelling a Domain concept. For a non trivial concept, you end up with at least 1 entity and some VO. Regardless of how many entities, VOs or simple objects will be involved, the important things are that you'll have a 'primary' entity (named after the concept) and other objects that work together to implement the concept and its required behaviour. Those together form an **Aggregate** and the 'primary' entity is the **Aggregate Root** (AR).

 The purpose of an AR is to _ensure the consistency of the aggregate_, that's why you should make changes to one only via the AR. If you change an object independently, the AR can't ensure the concept (the aggregate) is in valid state, it's like a car with a loose wheel.

 Before rushing out to name any entity an AR, first make sure you know the aggregate then make sure the AR really is responsible for the aggregate consistency. An entity acting as a container for other objects is not an AR. The consistency part makes it an AR.

 
## Domain Entity

 Modelling a Domain concept is more than coming up with a name and some properties. You have the name 'Foo' then what? If you start thinking about its properties or behaviour you'll end up with a data structure (resembling a table schema) with functions. That's because we think about all things that we want to persist and not about how the concept is defined and used by the Domain. We want to come up with a model fit for everything and that's the wrong approach.

 In order to define **Foo** you start with the use cases of **Foo**, better yet, implement those use cases as tests. This is how you identify the relevant properties and behaviour, that's how you design the entity. Once you have the interface defined, you can go wild with the implementation. In practice, you'll notice how you start with one entity and you end up with a whole aggregate. And that's how it should be, anyway.


