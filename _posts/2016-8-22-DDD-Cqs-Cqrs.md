---
layout: post
title: DDD Decoded - Modelling with CQS
category: DDD Decoded
---

While CQS/CQRS are a bit newer than DDD, they work so well together, one simply _has_ to use it. And I don't mean CQRS from an architectural point of view, but from a modelling point of view.
 
Simply put, when we identify a **business case** we ask ourselves: "Does this case has the **intention** to change the business state"? We're not talking about a particular state (objects, tables etc) but about business state as a whole (remember that in real world any change is implicitly persistent). If so, then it's a **command case** else it's a **query case** (this is CQS at work). 

Being in a query case means that you know you won't be changing anything and that the models will be read-only (this is CQRS, because we end up with specialised read **models**).    

But in a command case, things are a bit different. Many concepts can be involved however, usually only **one** will be represented by a **command model**. The other concepts will be represented by **read models**. We apply CQS for each concept and we end up with specialised command/query models for each, basically CQRS (not to be mistaken for a CQRS architecture, which is a different thing, but based on the same CQS principle).

As an example, let's say we have the `Create invoice` business case. Clearly, the business wants to retain the new invoice, so our business case will _change_ the business state. We've identified a concept too, the `Invoice` for which we need to find a **relevant representation**. However, because we're creating the invoice, it means we need a **command representation** i.e an [Aggregate](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-1). 

That's great, but there's more concepts involved besides the `Invoice`. We need to get the products and prices from an `Order`, we also need to calculate `Tax` and so on. Each of these concepts have a model as well, but because we don't need to change the business state for which those concepts are in charge, it means we're going to have **read models** representations. So, the `Order` model will be some read-only structure which contains all the data needed by the invoice. We always try to identify the most relevant model for that specific case, and not a generic model good for everything.

As a generic 'rule', consider the business case as being like a bounded context i.e it has a model which makes sense only inside it. Concepts are involved in many business cases, but we work only with their most relevant models and based on the action we need, either write or read, we end up with a command model or a query model for _each_ concept.

So, in a command case, we can find both command and query models of different concepts while in a query case, we're dealing only with read (query) models. Applying CQS early when modelling helps us to know upfront what types of models we're looking for, which gives us a bit more clarity and a productivity boost.   