---
layout: post
title: DDD Decoded - Domain Relationships Explained
category: DDD Decoded
---

One of the most trickiest things to understand in DDD is the domain relationship between 2 concepts. Most of the time when people think relationship they use the programmer mindset and they look for: `has-a` or `is-a` or `parent-child` or (worse) `one-to-many` etc  relationships. Which is valid when you write code but **absolutely wrong** when you do domain modelling.

What we're looking for is a **Domain Relationship** i.e how the Domain sees the relationship from a business point of view or, to be more precise, what is the level of association/partnership between the concepts and what business rules are involved in maintaining the relationship. And usually there are 2 types: **strong/always** and **weak/optional**.

# Strong (always) domain relationships

This one is the easiest: it simply means that in a given business case, a concept **always** needs the other. Examples: Customer and Order, Author and Tutorial etc. Contrary to how many devs think, it's never "Customer has orders", it's always an action about an `Order` which might involve or not the `Customer`. When the customer places an order, we need to create that order and we're going to use a specific `Order` aggregate. But orders don't just appear from thin air, a customer is _always_ involved so we can't create a new order without mentioning the customer.

`Order` concept requires at its creation a `Customer` so, in our Order model, we need to find the representation of the Customer that will be used by the Order aggregate. In most cases, we only need the simplest model that represents the _existence_ of the Customer, so we'll use its id. That id is a Value Object and will be a part of the Order aggregate. We don't care about a Customer aggregate, because we don't need one for this business case. The `Order` is the main concept, `Customer` is secondary. 

However, if we are in a "Cancel order" case, there is no `Customer-Order` relationship to care about (there is an `Order-OrderStatus` one, though). So, we can cancel the Order without involving/mentioning the Customer, but, as I've said, there's a different relationship to take care of now. 

Let's say that an Order _always_ needs to have a status. It might be tempting to include it as part of the aggregate but... if you look at it like the business does, it makes no sense for this _Value Object_ to be part of it. What makes sense is to consider the `Order Status` as being a sort of metadata associated with an Order. 

When the Order is created, the only place where this relationship should be represented is the resulting `OrderCreated` event. That's because a status like "Pending" is implicit, we don't need any particular input or business constraint for it. But we still need to 'announce' that the order has that status, because it is part of the business state change.
  
Now, when cancelling an order, we need to specify a different status for the order, however because we can have different outcomes based on the existing order status, we need to identify an aggregate just for that. This aggregate involves the [Entity](http://blog.sapiensworks.com/post/2016/07/29/DDD-Entities-Value-Objects-Explained) `Order` (via OrderId) and the [Value Object](http://blog.sapiensworks.com/post/2016/07/29/DDD-Entities-Value-Objects-Explained) `OrderStatus` and while being a model specific to changing the order status (representing the relationship itself) the resulting change is still associated with the `Order` concept.

Using Event Sourcing, this means that the `OrderCanceled` event is considered to be part of the `Order` and so, the entity's id is the order's id. But the aggregate is about the _relationship_ between `Order` and `OrderStatus`, it's not just an Order aggregate. If this sounds weird, I can only say that it will make sense with more experience.   

# Weak (optional) domain relationships

Basically, 2 concepts _can_ be associated together. Think `Post` and `Category`. Both concepts are independent, but they can work together. In the business case  "Assign post to category" we need to create the association between them according to business rules. The relationship aggregate will involve both concepts and we get an interesting situation, because we end up with 2 entities but we need only one to act as the aggregate root. Which will it be?

Clearly, we shouldn't just flip a coin. We need to understand the concepts better as well as the purpose of their association. It turns out that `Category` exists to organize posts, while a `Post` really doesn't care about categories. This relationship is more important (relevant) for the `Category` so, we select it to act as the aggregate root. This means that in the resulting event, the entity id would be that of the `Category` while `PostId` will be just a field.

Being a relationship, even if changes are considered to 'belong' to a specific concept, the aggregate is _always_ a model for the relationship.

An interesting example is deleting something. If I soft delete a post, what I have is in fact a new relationship between the `Post` and `Deleted` status, the status is never part of the post itself (as a concept) but always some metadata associated with the post. Btw, the status is a Value Object even if its implementation is just an enum.

Another example can be "Pay invoice". Yet again, a relationship between an Entity (`Invoice`) and a Value Object (`InvoiceStatus`) . The status is metadata for the invoice and we need a relationship aggregate to change the status. From the domain point of view, something associated with the Invoice entity has changed. 

# Conclusion

The hardest part about domain relationships is to properly identify them; many times, especially when an Entity and a Value Object are involved, we can think that the VO is part of the Entity's aggregate, when in fact it's a relationship that might need its own aggregate. But with more practice, spotting relationships will become second nature.  
 