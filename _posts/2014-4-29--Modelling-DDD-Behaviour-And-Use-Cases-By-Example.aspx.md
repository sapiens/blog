---
layout: post
title:  Modelling DDD Behaviour And Use Cases By Example
category: Domain Driven Design
---

Let's take the community's favourite example: modelling an Order. In this scenario we're talking about a very trivial model since it's a blog post not a book.

 First of all, what is the purpose of an Order for our Domain? In most of the cases the Order captures what a customer (C) wants to buy from you (products/services), coupon codes that the C want used with that order, a shipping address where the products will be delivered. I keep this simple, I won't involve billing here.

 Let's talk a bit about order lines (everybody knows that an order has lines). Each line represents a quantity of a product at a specific price. Since prices change often we really need to keep the price valid at the time C placed the order. But what is a Price? Note that now I'm talking about the domain concept of 'price' . A Price is pretty much always a positive (or zero) value and a currency. 10 is not a price, it's a number. -5 USD can't really exist while a simple decimal is again a number.

 Here it's important to ensure we've got the right definition. After all, you are seeing -5 USD (whatever currency) quite often on an invoice or receipt. But is it a Price? Or is it a _Discount_? Or maybe a loss? So let's define Price in code

  
```csharp
public struct Price
    {
        private readonly decimal _value;
        private readonly Currency _currency;

        public Price(decimal value,Currency currency)
        {
           if (value<0) throw new ArgumentOutOfRangeException();
            _value = value;
            _currency = currency;
        }

        public decimal Value
        {
            get { return _value; }
        }

        public Currency Currency
        {
            get { return _currency; }
        }
    }
```
  I've implemented it as a struct (a minor detail), however in DDD this is called a **value object** . The _Price_ is a domain concept that encapsulates values that will be treated as a single value (in lack of a better term). What's important here is no need for an id, we care ONLY about values and 10 USD is always the same as any 10 USD (as a value not as a concept).

 Back to our Order, can we have an order if 0 (none) products were ordered? Can you even say without sounding weird? An empty order is valid order? Not likely. The Order actually needs at least 1 product to be ordered so that the order can exist. This means that an Order is _defined_ by at least 1 order line. This is very important to understand: the Order doesn't _have_ order lines, it's **defined** by the order lines. No lines, no order, as simple as that.

 Here's how the OrderLine looks like

  
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
  OrderLine is a value object (VO) as well. This is a bit tricky as the OrderLine does have a kind of id, the _Position_ property (I could have called it Id). But that id' is valid ONLY inside the Order. And really, an OrderLine makes no sense on its own without the Order context. The lesson here is that not everything with an unique identifier is an entity, because it does matter the scope in which the id makes sense. The OrderLine will always be a part of the Order, everytime you retrieve an Order you'll get all the OrderLines. **Update:** [Is OrderLine an Entity or a Value Object](http://www.sapiensworks.com/blog/post/2014/05/20/Is-OrderLine-an-Entity-or-A-Value-Object.aspx)

 You see that I store the product id and not the whole Product. Why is that? The Product is a domain concept but the whole definition make sense in another context. Here I really need only an id, for this context this is the 'definition' of the product. We're modelling only things that make sense in the current context and we're reusing whole concepts only if they keep their meaning (definition) across contexts. For example, Price is the same concept regardless of the context.

 And speaking of general concepts here's another one: Coupon Codes / Giftcards code. Every customer loves these, after all this how they get a _Discount_ . I won't model a Discount in this post (it really looks like a Price ;) ), I've just mentioned it to show you how things come together using a Ubiquitous Language. We have discounts, not negative prices or just a negative decimal.

 In this example, the C can use more than one coupon code (I'm very generous) but with a catch. Each coupon needs to target a different aspect of the Order. So you can have a coupon targeting shipping (free shipping if if the order value is above a threshold) or quantity (10% discount rate for more than 10 items bought) or taxes (we pay taxes if the order is above 99$) etc. C can add only one coupon for each category, more than one are ignored.

 First, let's define the CouponCode

  
```csharp
public class CouponCode
    {
        public string Code { get; private set; }
        public CouponTarget Target { get; private set; }

        public CouponCode(string code,CouponTarget target)
        {
            code.MustNotBeEmpty();
            Code = code;
            Target = target;
        }
    }
```
  Yet another VO. This DDD is full of them, really! Note that again, we don't have just a string named coupon, we have a full domain concept defined here. Ints, strings, decimals etc are language primitives, not domain concepts. DDD is about modelling and using concepts. And the CouponCode is very important concept. Note that there's no behaviour defined, so how is a coupon applied? Not a part of this post, but the CouponCode will be applied when generating the invoice (by a Billing service).

 However, the coupon codes needs to be part of an order. And we have the business rule specified above (only one coupon per aspect). It's time to take a look at the Order class

  
```csharp
public class Order
    {
        public Guid Id { get; private set; }
        private List<OrderLine> _lines=new List<OrderLine>();
        public IEnumerable<OrderLine> Lines
        {
            get { return _lines; }
        }

        public Order(Guid id,IEnumerable<OrderLine> lines)
        {
            Id = id;
            lines.MustNotBeEmpty();
            _lines.AddRange(lines.OrderBy(d=>d.Position));
        }

       
        List<CouponCode> _coupons = new List<CouponCode>();

        public IEnumerable<CouponCode> Coupons
        {
            get { return _coupons; }
        }
       
        public void AddCoupon(CouponCode coupon)
        {
            //rule - only one cupon per target type

            if (_coupons.Any(c=>c.Target==coupon.Target)) return;

            _coupons.Add(coupon);
        }

        public Address ShippingAddress { get; set; }

        public void CancelLine(int position)
        {
            _lines[position].IsCanceled = true;
        }
    }
```
  A very simplified Order. Note that the Order really needs the OrderLines, passed as a constructor init value. This implements the fact that without an order line the order can't exist. But we're talking about coupons here. So let's focus on the _AddCoupon_ method which implements the business rule. This method is a **behaviour** of the Order entity. It _uses_ the CouponCode concept so it's also a **use case** of the CouponCode.

 Speaking of behaviours and use cases, there's another behaviour: _CancelLine()_ . You tell the Order that one lines should be cancelled and the implementation is a use case of the OrderLine. But even the Order itself can be part of an use case, for example when generating the invoice (you can't have an invoice without an order).

 You see that you can't do 'order.Coupons.Add(coupon)' or 'order.Lines.Cancel()'. It makes sense to let the Order add coupons since it encapsulates how/if a coupon should be added to an order. It also natural that you'd tell the Order what to do with one of its details. The Order contains the business rules that may allow or not the action, also, it's up to the Order to decide how an action should be implemented. It's just proper OOP here.

 You'd be happy to know that **Order, Price, OrderLine, CouponCode, Address** form together an **aggregate**. It's then obvious that the Order is the **aggregate root** (AR). But this came out AFTER we've defined the Order concept, as well a other relevant concepts. The point here is that you don't decide the aggregate (root) first, you just model the concepts/behaviour/use cases using proper OOP and the AR comes up by itself.

 Also, the AR is not just a container for order lines, it's a full domain concept _defined_ by other smaller domain concepts. I think this is the most common mistake with DDD, to think of AR just as a container for its children (I don't really like to use the semantics parent-children in DDD, it's too CRUD for me) when in fact, it's a bigger Domain Concept encapsulating business rules involving smaller concepts of the same aggregate (excuse the abstract language).

 In conclusion, you can do DDD easier if you're focused on modelling the correct domain concepts with behaviour, identifying and implementing the use cases in a proper OOP manner and all the DDD technical terms (some call them tactics) will appear obvious by themselves. This is exactly like using a design pattern, you don't say "Hm, what design pattern should I use today", you just try to code a solution to a problem then you realize you're using/discover a design pattern.


