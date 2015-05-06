---
layout: post
title: Don't Use ORM Entities To Model The Domain
category: Best Practices
---

Initially I wanted to include this in the Entities and Value Objects explained post, but I think this matter deserves its own post.

 When you model the Domain, for your own good FORGET about databases, ORMs etc! Because the design of the model must be dictated by the domain. If you put an id in a domain object, then it better have its use within the Domain. Keeping that code clean, will make the code much easier to understand and to modify.

 If it happens that a type will be persisted with an Id because that's what the ORM wants, you should NOT change the Domain model. Put the id in the Persistence Model and leave the Domain Model as it was.  If you change the Domain object just for the sake of another layer, in fact just for the sake of a different BC, that model becomes inconsistent within its BC. Yes an Id doesn't change much or making a property virtual (because that's what the ORM requires), but it tells to a future maintainer that it's ok to subclass the type and override a property, because well... that's why is virtual in the first place, right?

 What do you do, you put in the comments : "this is in fact a Value Object but we need the Id for NHibernate, DO NOT change!" or "This property is virtual because EF requires it"? Come on, you want to do DDD and have a robust code, or you just want to throw buzzwords around, while continuing to do the things 'the old way' ? If that app, switches to use Redis or MongoDb, an ORM becomes useless, but hey, your 'domain' objects are tied to the ORM requirements so you just keep them. Of course, now any (new) developer will the comments then asks himself: "Huh, we don't use NHibernate... Wtf is this?!"

 The objects , entities that you define for ORM use are for ORM use ONLY. Those objects must evolve according to the storage needs. If you used them for the domain, you'll have an abomination, the entity will serve two masters and none of them well. Any time you'll change one because one master needs it, you have to account for the effects from the other master. You know not to mix the UI with calling the db, why do you think is ok to use a Persistence implementation detail as the base for your Domain?

 "It's about DRY" you might say. I like DRY, but here you just invoke it as an excuse for mixing responsibilities.  You don't have identical code in two layers, they might 'share' 90% but they are different and those differences matter. Those will give you the headaches. Even if at one point they are 100% identical: each code was written with **DIFFERENT INTENTION** in mind. It just happened to be identical. If you keep them separate, then one can change without messing the other.

 What about maintainability, you ask? This is maintainable code. The code you can understand in a second and you can modify in a minute. If you are worried about writing the same properties, there is an old technique which usually is abused but here it's very appropriate: it's called copy/paste and it will solve this specific problem in a second.  
  
 What's more difficult: to copy/paste in 2 seconds every time it makes sense or to keep one code base and then praying that modifying something didn't break the functionality in another layer?

 That's why I'm a firm [supporter of the separation](http://www.sapiensworks.com/blog/post/2012/04/07/Just-Stop-It!-The-Domain-Model-Is-Not-The-Persistence-Model.aspx) of the Domain Model from the Persistence Model when using an ORM. The ORM docs might say that you model the domain entities, but in fact you'll be always modelling the Persistence Model because this the ORM's purpose: to handle the persistence of objects in a relational database. It's not its purpose to model domain behavior.

 Model the Domain entities for the Domain needs, model the Persistence entities for the Persistence needs. Don't forget that an ORM is ALWAYS [an implementation detail of the Persistence](http://www.sapiensworks.com/blog/post/2012/04/15/The-Repository-Pattern-Vs-ORM.aspx).


