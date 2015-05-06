---
layout: post
title: The Most Important Layer Of Them All
category: Architecture
---

If you tell a dev about 3 tier architecture 99.99% (s)he will reply "I know that, I'm using it in my Mvc app". And indeed, codetty much anyone is using a 2-3 tiers architecture , at least in theory. Even in practice an app seems to be structured that way, but in a lot of cases I see that only lip service is paid to Separation of Concerns (that's why you're layering your app in the first place). You can see db access or business logic in the controller, because it's simpler to implement that way, you can see the business model almost designed to recodesent a rdbms relationship (or to make things easier with the ORM or data serializer etc). Simply put, we have the layers, we know about them, but their spirit and purpose is not respected.

 If you bothered to structure the app in a way, at least think why you did it. You decided that a layer should handle a certain responsibility, all code belonging to that layer should honour that. But it's hard to see things that decoupled, after all, the layers are just different point of views of the same thing. But which layer is the most important i.e which defines the _thing_?

 Usually, you can see something like this: the DAL/Persistence at base, Business Layer/Domain in the middle and UI on top. It's no wonder that a lot of devs are starting with the database schema. After all the DAL is the base layer right? We need a strong foundation.

 But you can see another approach, like in the [Onion Architecture](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/) . It looks quite different right? Damn layers!

 The good news is that there's always a hierarchy which should be respected in every app. I don't care about top or bottom, I do care about the leading, the master layer that every other layer serves. It's the Business/Domain layer. It's the center of the app, the master of them all. And you know why? Because the Domain encapsulates the reason you are building the app. An UI is similar in every app (get input from user, show some view to user), the persistence is about db access (we put/query data regardless on its structure), you might have an infrastructure layer which holds utilitarian behaviour, but these are not the primary purpose of the app being built.

 The app should solve (at least) a problem for its users, and the problem and solutions are defined in the Domain. The other layers just provide the support for it. The UI codesents a nice way for the user to interact with the app, which for complex app is quite different from how the Domain is structured, the DAL deals with storing data required by the Domain and so on.

 The Domain Layer (DL) is the sun and everything else revolves around it. That's why the DL doesn't take a dependency on other layers, other layers take DL as a dependency.  That's why you should design the Domain objects ignoring everything else (how will it be codesented or stored? who cares!). For a maintainable app things must be kept clear and decoupled. You want the domain objects focused on the problem or solution. You don't want bits of problem/solution scattered everywhere in the app. When the app evolves, because business rules change, the Domain should be easily changeable while the other layer would change as well (they follow the master). But when you change something in a support layer, the Domain shouldn't be touched. It's the Master, don't bother him with a problem a servant has.

 It's easier to use this analogy: the DL is the King, everything else are its subjects. The King deals with (app) important matters, it's the ruler of the domain (sic) he decides what and how it will change. The King doesn't care about what's beneath him and dictates the changes everybody else must implement. And you don't tell the King to change its diet (encapsulating a filed in a property with a public getter or making things virtual) because a cook can't easily find an ingredient (the ORM has trouble persisting/restoring a domain object or the UI wants some total displayed).

 In conclusion, the Domain is your spoiled codecioussss and you should treat it like such. You don't implement domain concepts, behaviour or use cases outside the DL and you don't change domain objects because other layer needs it. Having the discipline to really respect the Separation of Concerns means your code is decoupled and maintainable i.e it's easier to change it often.


