---
layout: post
title: Rich Domain Doesn't Mean Fat Objects
category: Domain Driven Design
---

Let's look at a CRUD app. Is it a business app? Yes, because it still provides business value. Sure, the business just wants to store and retrieve some data but that data does have business semantics and usefulness. Do we have business rules? Yes, we do, data validation is implementation of business constraints.  

But what about queries? What about calculating how much you owe in unpaid invoices? Sure, it implies just reading some data but the calculation is done according to business rules i.e it is business behaviour. A CRUD app does have business behaviour and quite plenty of it, however what makes it simple it's the fact that the behaviour is quite straightforward and thus, easy to implement.

And yes, a CRUD app can still benefit from DDD, the strategic part aka understanding the business. After all, this is how you know you're dealing with a simple domain. But the tactical patterns are pretty much useless in this case.

So how does a rich behaviour domain looks like then? Well, not only that it has more concepts and use cases, there's a lot of  **decision making** based on business rules. And this is the hard part. We need to untangle the decision-making process.

Thing is, it's not about having big fat objects. The OOP fear of an anaemic domain shouldn't drive us into the other extreme, where we put everything related to a concept into one object. We need to model the complexity of making decisions and maintain a consistent business state. And from an implementation point of view, we have way more services (which are pure behaviour) than aggregates. Yes, an aggregate encapsulates _some_ business rules (constraints) but only those required to ensure consistency when a change is made.

However, the decision-making part which contains the bulk of rules should be implemented as services, because this is what they are: business behaviour with no state of their own. Basically, you end up with a number of concepts (and their encapsulated limited behaviours) and lots of services. This might look like an anaemic domain but it's not.

It's easy to think "I'll just put all related User business rules in the User object", but it's the wrong design which complicates things in the long run. While in OOP we want to keep together data (state) and related behaviour, the methods should manipulate or process that state only. In DDD the aggregate is a consistent group of other objects (concepts) treated as one (bigger concept), exposing a facade (the Aggregate Root) which will ensure that all changes maintain the unit's consistency.

But that's it! We don't add other behaviour just because it _is related_ to that concept. We don't want a fat object that acts as container of functions. We represent a business concept in code. Its use cases aren't part of its definition. When trying to decide where you should put a behaviour just look at how the business uses the concept.

As an example, I have the concept of "Chair" . If I have a use case where the chair is moved, most of OO programmers will have something like this `chair.Move()` . Which is wrong, because it's not the Chair's responsibility (from a business point of view, we always care about the business point of view)  to handle position and movement. It would be an external behaviour (another aggregate or a domain service) which will have a `.Move(chair,location)`.

In conclusion, I would say that a rich domain proper implementation depends a lot on understanding how the business thinks and less on programming principles and patterns or technical implementation specifics. The code should serve the business, the business shouldn't adapt to code.  


  