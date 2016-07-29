---
layout: post
title: DDD Decoded - Entities and Value Objects Explained
category: Domain driven design
---

One of the staples of DDD _mindset_ is the partitioning of **business concepts** into: _Entities_ and _Value Objects_ . Notice, that I've said _concepts_ not objects. The difference is that an object _might be_ an implementation of a model of a concept. It's not about how we write code, it's about how we 'label' a business aspect, in a more developer friendly manner.

# Why do we need these tactical patterns?   

One practical advantage of DDD is that it tells us how to organize domain information in a way that we can implement it easier. Once we know we're dealing with an _Entity_ or a _Value Object_, we get some hints about how to continue modelling or how an implementation looks like. 

# Entities are concepts whose instances are uniquely identifiable

Actually, it's a bit more: yes, they all have one _immutable, read-only_ aspect or detail (call it however you wish) that acts as an identifier, but the thing with an entity is that we can change everything related to it (except its id) and it remains the same instance. It sounds a bit abstract and dry, but things are usually simple. It's quite easy to identify a concept that is an entity: the domain expert will refer to it as a single, identifiable item. _Customer, Invoice, Order, Article, Post_ etc are examples of common concepts which are entities. As long as the business care about tracking each instance of that concept, we're dealing with an Entity.

In many cases, you get the hint directly: the domain expert will mention a _unique number_ or _unique id_ or _serial number_ etc as being part of that concept definition. Even more, when we identify models for business cases which change the instance of that concept, we keep seeing the identification detail mentioned every time.

## What being an Entity tells us

The most important hint is that an entity designates a 'bigger' concept, therefor we're dealing with an [aggregate](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-1) and we need to 'dig' deeper in order to find out its _composition and its business rules_. 
Another thing is an entity can't part of an aggregate of another entity. But we can have _references_ (not the best word) that is, an aggregate can 'point' to another entity. Maybe a better way of saying it is that when we have a **domain relationship** between 2 entities and one 'needs' the other one, the expression of it is a reference a.k.a the id of the other entity.

An example is an Order is always placed by a Customer, so the Order entity has a relationship with the Customer concept, in the sense that we need to represent the Customer as being part of the Order (Create) aggregate. But since Customer is an Entity, only its id will be part of the Order aggregate.
      
# Value objects are simple or composite values that have a business meaning

A reminder that early DDD was mixed with OOP, a better name for the _Value Object(VO)_ would be a Value Concept. But the name stuck so we keep saying Value Objects even if its implementation is not an object. VOs usually are not treated with the same respect as the Entities, however they are equally important. It's not that hard to identify a VO, it's a concept that doesn't have an explicit identity detail. Actually, we can say that the identity of a VO is represented by all the values that VO has. Change something and it becomes another value.

VO not only have values, but they incorporate business constraints that the values needs to respect. Basically, a VO is a 'smaller' concept, which is part of a (big concept) Entity.  Most VO are small enough, 1-2 values but we can have bigger VO which combine other VOs.

Let's take an example: A **Price** is a concept which is represented by 2 values: Amount and Currency (itself a VO). It doesn't have an explicit identity and most importantly, the business doesn't differentiate between one $5 and another $5. 

Another example is an Order's ordered product, a detail of the Order Entity. And no, I haven't seen any business talking about 'OrderLines', but they talk about an order's _Items_. The VO `OrderItem` is a composite of 3 things: _Product_, _Quantity_ and _Price_. As a concept, `Product` is an entity and a VO can't contain Entities so what we'll have is a _ProductId_ (or SKU) which itself is a VO. And it might seem that `OrderItem` has an identification detail, the _product_, used to identify a specific item from the order and making the item look like an entity, but the important thing is that `OrderItem` is a component of the Order, a minor concept which isn't used directly outside the Order. A business can't identify an `OrderItem` only by its 'id', it always needs the `Order` itself to act as a context for that 'id'. That's what makes it a VO, a true entity doesn't need another entity to be identifiable.    

Sometimes is tricky to identify the nature of a concept, it might look like an entity, however if it represents a minor concept which make sense only as part of another Entity, then we're dealing with a VO.     

## What a VO tells us

* Business Constraints - a VO communicates business semantics and usually this means those values need to respect business rules. A VO is **always** valid.  
* Immutability - due to its nature, a VO never changes. It can't change or else it becomes another value. So, set its value(s) once and that's it.
* Usually a component of an aggregate - Business functionality deals with Entities, it doesn't care about a stand-alone VO instance. Ideally, all components of an aggregate are VOs.
* Semantics > Structure - `Price` has an amount and a currency. `Discount` has an amount and a currency. Are they the same concept? No, and the business cares about that. Even if 2 VOs have identical structure and business constraints they aren't the same and they should be implemented as different things. DRY doesn't apply here, never reuse VOs! 

These hints make it easy for us to implement a VO and personally I go for an object with business rules enforced in the constructor plus getters. We can have methods but those methods should never change the VO, you can only create new VOs.

In some cases, the VO implementation can be just a simple enum (think OrderStatus: Pending|InProgress|Completed), no need to complicate our life.


# When CQRS is involved

While the nature of a concept is the same, regardless if it's a Command or a Query, these patterns make sense only for the Command model. A query model is just some (read-only)data which can span multiple concepts. While we can use some VO for queries, it's mainly a DRY convenience. 

# Conclusion

It helps to organize the business concepts using these tactical patterns, as long as we identify things _properly_. The most common mistake I see is devs who point to a data structure full of primitive types and say:"I have this Entity". Do not confuse the nature of a concept with a possible implementation of one of its command models.







