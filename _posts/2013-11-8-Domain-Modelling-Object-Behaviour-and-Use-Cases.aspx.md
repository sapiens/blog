---
layout: post
title: Domain Modelling: Object Behaviour and Use Cases
category: Domain Driven Design
---

One of the most trickiest but important thing when domain (OOP) modelling is to properly identify the entity's behaviours and use cases. But in many cases what seems a behaviour is actually a use case. It's quite easy to fall into the trap if you're not taking enough time to understand the domain.

 Picture a mobile phone, a 'dumb' one. To call someone you just type the number and codess the 'Call' button. In code it looks like this

  
```csharp
phone.Call("some number");
 phone.EndCall();
```
  Very straightforward. It's clear that "Call" is a behaviour of the phone. It's part of its intrinsic functionality.

 How about a smartphone? Well, first you start the Phone app then type the number then tap on the "Call" icon. Here's some code

  
```csharp
var dialer=phone.StartApp<Phone>();
 dialer.Call(""some number);
 dialer.EndCall();
```
  Well, things changed... Calling someone is no longer a phone behaviour, it's a phone use case. Starting an app is the phone behaviour, while calling becomes the Phone app behaviour. With a smartphone, codetty much everything is an app. What the phone does now is run apps. All you can do is tap the screen and codess the hard buttons which act as a controller sending some signals to the OS or the running app. But all the actual behaviour you want from the phone belongs to its apps. The phone is just the platform now, the most used behaviours are: tap and charge the battery.

  
```csharp
public void Charge(Charger charger)
 {
    charge.Charge(this.Battery);
 }
```
  The funny stuff is that charging is a use case of both battery and charger but encapsulated into a phone behaviour. Now, how much fun would you have if your phone couldn't be charged just by plugging in a usb cable or via induction? You would have to remove phone case, remove the battery and somehow connect it to the charger. And the phone would be unusable until the battery would be put back. See? If you remove the Charge() behaviour, the phone becomes much harder to use and even useless while the battery would charge. Removing that behaviour greatly affects other object functions and use cases. Having the "Charge" functionality enables the smartphone to be the device it is. The same is for tablets or other similar products.

 Let's say Jim is a landlord. He can charge rent in the sense that he has the right to demand rent. But the actual calculation and collecting can be done by other people too. After all, if  Jim owns 5 BIG appartment buildings and 10 other properties, I doubt that he will personally handle all the rent situations. It could do it, but this behaviour can be delegated. And Jim doesn't lose his landlord status because of it. Just as Jim could clean the properties himself but he has specialized staff hired for that purpose.

 The main point is that the Domain concept matters a lot in identifying object behaviour. The concept of Landlord is about ownership, but in a certain Bounded Context we may only care about calculating rent. While it seems something clearly suitable for a landlord,  the fact is your code isn't about ownership, it's about finance issues and it should reflect that. And instead of the Landlord you'd probably have a RentCollector or RentCalculator concepts.

 Let's consider an (e-commerce) Order and an Invoice. Is the creation of an invoice a behaviour of the Order? After all, it has the data the invoice needs. But it's not an Order behaviour, because the concept of an Order is basically a 'document' detailing what a customer wants to buy from you. An invoice is another document which uses some data from the Order but also other data (discounts rules based on volume or customer status). An Invoice generation is a use case of the Order.

 How about tracking the Order's fulfilment? We need to know what products are ready, what needs to be ordered from the supplier etc. All these together form another use case of the Order. How about the Order status (Pending, InProgress,ReadyToShip, Shipped,Delivered etc)? Does this belongs to an Order? It doesn't. The tracking of the Order's state isn't part of the Order concept. We're using additional data and behaviour to recodesent different use cases which are useful for the business and for the customer but which don't define the Order concept.

 There are a lot of Domain entities and Aggregate Roots which don't have behaviour but are involved in many use cases.  A domain object should model a Domain concept regardless of complexity or number of behaviours. The purpose is not to come up with big objects with lots of methods. The purpose of DDD is to recodesent the relevant business concepts, simple or complex.

 If you model the correct concept **as it is, not as you think it should be** based on technical rules or recipes, you'll have a very organic, flexible and thus maintainable Model.


