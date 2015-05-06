---
layout: post
title:  Why You Should Always Use CQRS with DDD
category: Domain Driven Design
---

Yes, I said **always** and I'm very aware of the gravity of the situation. It's not prudent to use absolute words but I'm ready to prove that "I'm Right" (tm).

 Modelling a domain is a tricky task. So tricky that you don't need additional concerns such as: how will it be persisted, how is it shown? Wait, on that page I also need some other info, should I add it to my model? I'm using that ORM, which requires to have a parameter less constructor or virtual properties/methods should I care about this? I'm using a RDBMS and I want my data to be denormalized but this complicates a lot the loading of this aggregate root, should I take this aspect into consideration?

 There is a reason there are (and you're probably building) n-tiers apps. You are structuring your app at least in 3 layers: UI, Domain, Persistence and you're doing this so that you can change anything inside a layer implementation without breaking the other layers. It's called decoupling and it's a trendy buzzword. It helps with maintainability, another trendy buzzword.

 Decoupling means each layer is it's very own domain (unrelated to the Domain), a context with boundaries (sounds familiar?) which has its very own model. Of course, the Domain model resembles the Persistence model and resemble the UI model. They are quite similar and in some cases they are even identical. But they belong to different concerns and thus they model different concepts and different use cases. So, in order to achieve maintainability is imperative to consider each layer autonomous so that you can focus on designing the right model for that layer.

 When you model the Domain, of course you should care ONLY about the Domain's concepts, behaviour and use cases. You forget there is any Persistence, UI etc. It doesn't matter how things are saved/loaded, it doesn't matter the UI needs a Fullname formatted in a certain way. Whatever doesn't belong to the Domain, stays outside it and nobody cares about it. You care about it when you're working on the relevant layer.

 So we have a nice, rich Domain model. You have a user friendly UI model. These 2 resemble but are not quite the same. One has business rules and uses codetentious words, the other is more straightforward and sometimes it combines different aspects of the Domain.  Even more, you might need different UI models for different purposes, one for the normal users and other for the company managers which love reports which use aspects of the Domain never shown to the other users.  And there you have Persistence. It has the ingrate role to try to please both Domain and UI.

 [CQRS](http://www.sapiensworks.com/blog/post/2013/05/04/CQRS-Explained.aspx) says: how about instead of having one model burdened by both business rules and query use cases we have at least 2 models, one for domain usage and one for query usage? The Domain can model complex concepts and behaviour. The UI just needs data which many times spans more than one domain concept. CQRS comes up as a natural solution: keep the Domain model relevant ONLY to business and use a different model for things outside the Domain.  Yeah, I know CQRS is about writes and reads, but the Domain is the only one which should do the writes while the other layers are concerned about reads.

 The Persistence is a special case since it has to deal with both write and reads, but the CQRS principle applies as well. Store the DM so that it can be easily restored and generate and update a read model suitable for queries. However, it's not the Persistence who creates the read models, but a service who then tells the Persistence to save that data. Of course, the Persistence is free to actually store things anyhow it pleases, what's important is to retrieve the proper model when asked.

 Once you know the  DM is 'safe' and doesn't need to care about UI or Persistence details, you can focus only on it and your life becomes much easier. And once you're done with the Domain, you can focus on the other issues.

 CQRS gives you the greatest benefit when you're doing DDD: it frees you so you can deal only with the actual Domain modelling while providing a solution to satisfy the other layers' needs. The way CQRS is implemented is another story and that depends on the app needs and on how comfortable or experienced the developers are with certain techniques.


