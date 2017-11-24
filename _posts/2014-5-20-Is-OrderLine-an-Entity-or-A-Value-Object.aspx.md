---
layout: post
title: Is OrderLine an Entity or A Value Object
category: Domain Driven Design
---

A reader left this comment on my post about [modelling an Order](http://www.sapiensworks.com/blog/post/2014/04/29/-Modelling-DDD-Behaviour-And-Use-Cases-By-Example.aspx) .

 
> Your order line is a Value Object (VO), you said, but VOs are, as you might know, immutable. How comes that IsCanceled and Position are settable?
> 
>  
 Let's see the Orderline again

  
```csharp
public class OrderLine
    {
        public OrderLine(int quantity,Guid productId,Price price)
        {
            quantity.MustComplyWith(q => q >= 0, "You can't have negative quantities");
            Quantity = quantity;
            ProductId = productId;
            Price = price;            
        }

		public int Position { get; set; }
        public bool IsCanceled { get; set; }
        public int Quantity { get; private set; }
        public Guid ProductId { get; private set; }
        public Price Price { get; private set; }
    }
```
  Not the greatest or the most flexible model, but this is what you get when you're using a fictional domain instead of a real one, you're inventing a context to match your model. In real life DDD it's the opposite.

 Anyways, the point here is not that we have a mutable VO, but why I've considered the Orderline a VO instead of an Entity? An object is not a VO because it's immutable and it's not an Entity just because you have a property Id (similar a class named Repository is not really a repository). We need business or technical reason.

 In my example, in fairness I don't need the Position property at all. Orderline #4 doesn't tell me anything meaningful and I can change the position without any business side effects. The question is do I really need to uniquely identify an Orderline? Can my order have duplicate orderlines? It really depends on the domain, there's no hard rule.

 Usually we care about the product and quantity. But when your domain rule says: 1 item per orderline only and you have 2 identical orderlines then what? Canceling one means cancel any of these 2  orderlines. You really don't care which is. Do we need an identifier for that?

 It's quite simple to slap an id on everything and declare it an entity. It's the easy way out IMO.  After all, some things are not just values and if it isn't a VO then it's an Entity right?  Perhaps, but if you don't really have a business reason to uniquely identify an object should we still call it an Entity? Does the business cares about Orderlines? I don't think the concept even exists. The business cares about what quantity of what items a customer ordered.

 A customer never cancels an orderline. They cancel the whole order or a product or (if allowed) they change the quantity of an ordered product. The truth is the Orderline is a construct to group together at least quantity, product and price. It's a way to structure an Order. But it makes no sense outside the Order.

 Let's assume that things can be shipped independently. I had Amazon combine one full order and a part of another in one shipment. Does this means I have a valid reason to make the Orderline identifiable? But what if I order 3 copies of something and 2 come in one delivery and the third is shipped with another order? How do I track 2 deliveries for the same orderline? Complicated stuff!

 Maybe we should see things in a more natural way. We didn't ship the orderline with id 245, we shipped 2 items part of the order #28890-3349 . Inventory and Delivery care about quantities and products, Billing cares about quantity, product, price, discounts. None is really using the orderline concept. And the invoice usually has more lines than the order.  Look at the Amazon invoice, it contains the order id and the list of delivered items. The invoice is generated right before the items are shipped and only for the shipped items.

 Every use case involves product, quantity or price but not the concept of Orderline. So why is there a need of an id for it?  I understand why it's so easy for many people to consider the Orderline an Entity: you want to identify a line of the Order. But if the business language doesn't use the Orderline concept (why lines, maybe there are columns), then let's consider it some technical detail used by the Order object. I've really never heard someone saying: "I have some orderlines to ship for this client". It sounds pretty weird, too. But "I have 4 copies of book X to ship for this client"  sounds more natural.

 That's why I can't consider Orderline an Entity. It isn't used as an entity by the business, it's used as a group of values. Every use case cares about some detail but not about the orderline as a concept. Billing, Shipping don't care about a certain orderline, not even the Order itself cares about a line's identity. I can have 1 item per orderline or a quantity property. When the client cancels or change the quantity, an orderline can be changed or some orderlines get canceled. But these are internal technical details. It's the way I've chosen to represent that information. The business behaviour and use cases don't care about it.

 Shipping items A and B of order #32438 is not really the same thing as shipping order lines #2 and #3 of the same order, because only the Order knows about order lines. Business and clients know about products and quantities.

 It's true that you can't easily fit the Orderline in the Value Object definition especially with mutable properties. In conclusion, I would say the OrderLine is neither an Entity nor a Value Object, but a POCO. And regardless of how you'll call it, the class will look the same. It's an internal detail and it's proper OOP. This is what matters. Personally, I care very little which is which, if it models the proper business concept in correct OOP form, this is all I need.

 P.S: I always tell people to just model stuff according to business view and worry later about how things are called according to DDD terminology. The point is we are using DDD to design a better app, we don't create an app for the glory of DDD (to showcase buzzwords). 


