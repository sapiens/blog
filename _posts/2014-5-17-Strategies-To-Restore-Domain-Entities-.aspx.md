---
layout: post
title: Strategies To Restore Domain Entities 
category: Best Practices
---

Some time ago I wrote about ways to [restore a domain object from persistence](http://www.sapiensworks.com/blog/post/2012/04/06/Optimum-Ways-To-Restore-A-Domain-Object-In-A-Repository.aspx) . Today it's time to revisit the issue as things change and my opinion changed as well.

 Let's start with something easy: a data structure or an anaemic domain. Since there isn't much encapsulation going on this is the simplest and the most straight forward scenario. You can use any ORM or document db without any special setup, exactly like in tutorials. ORMs and NoSql both love data structures. Nothing to see here, let's move on.

 Things become complicated when you have properly designed business objects with encapsulated behaviour, especially aggregate roots since they can encapsulate a lot of concepts and behaviour (read: has many children). Why is that? It's because the way automatic tools work. Both ORMs and json serializers used by doc dbs are using reflection to populate the properties. If the property doesn't have a setter or it's encapsulating access to a more complex object then it won't work without some [black magic workarounds](http://lostechies.com/jimmybogard/2014/05/09/missing-ef-feature-workarounds-encapsulated-collections/) .

 Or take the easy way out and break encapsulation. It seems that you need to choose the lesser of two evils: corrupt your business object with persistence concerns or entangle yourself in workarounds which may or may not work for every object. And try to not forget to use the workaround every time you're adding a new object or change one.

 But there's another option, quite simple but boring as hell: memento or initialization values. The object's constructor (or a factory method) gets the memento as a parameter and then it's up to the object to restore itself. The object doesn't know about persistence details, it knows only about the memento, a DTO. And automatic tools love DTOs, no workarounds and fancy settings needed.

 The drawback is obviously the fact that you need to maintain another object, the memento, which looks almost identical with the business object. Copy/paste FTW! Pretty crude but effective. And it allows to decouple the business objects from whatever data structure needing to play nice with the ORM or doc db. Btw, I'm assuming the CQRS is used and we're talking here only about persisting the entities for domain needs. The read model should be generated from the object itself, the memento might be used but it's not intended to be used as a read model.

 I would recommend the memento only for important domain objects that you want to keep them properly encapsulated and you can't find a decent compromise for the automatic tool. The memento does add to maintenance, it has to worth its cost.

 There are a lot of badly designed (modeled) business objects in the wild. A lot of them because the developer still uses a CRUD mindset or because the object has to be compatible with the persistence solution. First, we need to make sure that we have properly modeled the domain objects (ignoring the database all together). Then we can decide on the persisting strategy. RDBMS by default or ORM by default are just the lazy easy out. It can work provided you haven't a rich domain. If you have, then you have to weight your options carefully and this means choosing both the right storage and the way the objects are restored.

 Almost forgot... Using a memento does have another advantage: it's easier to test the object. For some tests you need to start with a certain state. Initializing a default object then setting properties or calling methods until you get to the desired state isn't the best approach, especially if the object generates domain events. Using a memento is the cleanest way to do it.

 In conclusion, there's no best way to restore **every** object in **every** app. You have to decide on application basis and sometimes even based on object's complexity. Mix and match approaches as they fit the objects, you want clean code and shoving everything in the same db or with the same tool (ORM) will just complicate your life in the long run.


