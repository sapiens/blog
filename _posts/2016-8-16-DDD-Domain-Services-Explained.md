---
layout: post
title: DDD Decoded - Domain Services Explained
category: DDD Decoded
---

Domain services are a bit confusing at first, when you don't know exactly what a an Application Service is. Also, many developers try to cram a lot of business rules in their [Aggregates](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-1), when a Domain Service (DS) would be more appropriate. But let's start with the beginning.... 

# Why do we need Domain Services?

Some business rules don't make sense to be part of an Aggregate. For example, you want to check an account balance before debiting an account. Or you want to see if a certain operation is allowed by the business rules (not to be mistaken for user authorization). Or you want to perform a simple calculation according to domain rules. Basically, any business rule required to move forward a business case, which doesn't belong to an aggregate should be a Domain Service. This is the DDD term for **business behaviour** outside an aggregate or a value object.

# How to identify a Domain Service (DS)?

The easiest way is to simply check if the rules are having to do with business constraints required to maintain an aggregate invariants, including its value objects. If they're not, they're a DS, even if it seems related to that aggregate somehow. Many times you need to generate a value object that will be part of the aggregate. For ex: you need a `Tax Calculator` behaviour which will generate a `Taxes` value object which is a part of the `Invoice` Aggregate. 

In other cases, you might need to allow an operation. Remember the [transfer money example](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-2)? Wha if we have a rule that says that you can't debit the account id the amount is lower than the balance? In this scenario we're actually dealing with 2 Domain Services: one is used to calculate the `AccountBalance` Value Object (VO) the other encapsulates the "amount must be greater than the balance" rule.

# What about the Anaemic Domain anti-pattern?

In a business case we can use multiple DS and this might look like the Anaemic Domain. But it's not, the reason being that Aggregates should be as small as possible, not because someone said so, but because they should contain **only the relevant behaviour** for that model. Not _every_ related behaviour. If something is 'outside' an Aggregate, then it's probably is a Domain Service.

# Implementation tips

A DS is just a name signaling business behaviour. It doesn't mean you have to implement it in one way or another. In many cases you can **implement** all the domain services from that [Bounded Context](http://blog.sapiensworks.com/post/2016/08/12/DDD-Bounded-Contexts-Explained) as (static) functions in one class, while in other cases you might want one class per DS. As with everything, it depends.     

A DS should be visible and consumed inside that Bounded Context only! Yes, you can define interfaces that can be used by your application service, they're great for testing, however their concrete implementation should be in the same BC as the business cases where it's used.   

## External services

Let's say we need to calculate tax but the domain expert says they're using some website to do it (and luckily for you, it provides an API and a client library). We need that functionality, but it's implementation is not part of our Domain. This is what I call an External Service. The easiest way to use it is to define an abstraction (interface part of your BC) that will act as if it's a Domain Service, however its implementation will be part of the Infrastructure and it will act as a wrapper around the client library. This is how we keep things decoupled. If later the calculation will be performed in-house, just implement a new class inside that BC and reconfigure the DI Container. Simple stuff.

What if we need a service which is part of our app but part of another BC? For a monolith, you can cut corners and use it directly (assuming you weighted in the consequences). For a distributed app, I'd suggest the External Service approach. In the end, for the BC it's something outside its boundaries, it doesn't matter where the actual functionality is implemented as long as it's not inside the boundaries. For the BC's point of view, it's an external service.

# Conclusion

We have Domain Services simply because we want to keep a concept's model relevant, so any behaviour which doesn't _naturally_ fit that model needs to be expressed somehow. Also DS shouldn't care about state, they represent domain behaviour only (their implementation should be stateless).    