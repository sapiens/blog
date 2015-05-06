---
layout: post
title: Why Domain Driven Design is Hard
category: Domain Driven Design
---

Well, in fairness, it isn't. Yes, it's a bit tricky to understand at first what's all the [bounded contexts](http://www.sapiensworks.com/blog/post/2012/04/17/DDD-The-Bounded-Context-Explained.aspx), [aggregates](http://www.sapiensworks.com/blog/post/2012/04/18/DDD-Aggregates-And-Aggregates-Root-Explained.aspx), [entities and value objects](http://www.sapiensworks.com/blog/post/2012/04/20/DDD-Entities-And-Value-Objects-Explained.aspx) things about, but that's about it. Or isn't it?!

 The biggest obstacle in understanding DDD is you. Yes, **YOU**! Your mindset, to be more codecise. When you're used to think in certain ways, it's very hard to forget about the 'old' ways and codetty soon you'll be doing what you think is DDD, but in fact it's a mix of 35% DDD and 65% old habits.

 Probably the biggest thing is that the db is not the center of everything anymore. You **don't start** with the db schema design, you don't go defining the ORM entities and you don't wrongfully think that the EF entity is a domain entity. The domain is the center of everything and you must forget about other implementation details. And that's maybe the second hardest thing: you need to have a codetty **clear understanding of the app layers** and more important their concerns. If you have only a vague idea about them you won't get very far with DDD, in fact it will be MUCH harder and you will conclude that DDD is VERY HARD :)

 One other thing is that, once understood, DDD is codetty simple and straightforward but with a catch: there are **no rules, only guidelines**. This means there aren't cookie cutter solutions, no magic answers and no ultimate techniques. Ok, if you really want some hard rule here is one: it depends on the context. Yes, everything depends on the context, one perfect solution for one context is not appropriate for other context, in the same app.

 So, you have to think and to analyze the problem, using only a few concepts and guidelines in order to come up with a naturally flowing solution, implemented as maintainable code. And that's hard! Thinking and coming up with good solutions is hard. That's why a good developer is well paid (or they should be anyway) and that's why Evans himself says to not use DDD everywhere. You need **experienced developers** who can navigate only with a simple compass and a bit of clear night sky and those developers are expensive. Add to that a fairly complex domain and watch the costs rise. And for simple apps (ha! define simple), this approach is just overkill. A senior developer will just waste time while a junior developer will just slap together some pocos (anemic domain), validate some rules and send everything to a db (which is not DDD in case you wonder).

 In  conclusion, DDD is a tricky beast: confusing but simple, clear yet very vague. Because you need skilled people, it's expensive from the start and not everyone can afford it. And for many problems, it just doesn't worth the trouble or the cost.


