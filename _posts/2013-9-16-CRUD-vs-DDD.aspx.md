---
layout: post
title: CRUD vs DDD
category: Best Practices
---

There's a lot of CR(eate)U(pdate)D(elete) apps (especially web apps) in the wild and even worse, the CRUD mindset is prevalent. Even most those fancy OOP, wanna be DDD full of design patterns apps are actually built with that mindset. And then people complain that programing is hard and that entity is huge and scary to modify.

 There's a time and a place for each tool so let's see in what scenarios the CRUD approach make sense.

 If you want to get some data from users, do minor processing, save it to a database then read it back to present it to the users, that's the actual definition of what a CRUD app is. A simple, data driven app who serves little more than as an UI for the database. You don't really need fancy stuff, TDD, design patterns, Dependency Injection etc. You only need to create the required data structures and that's it. A (micro)ORM is extremely useful here, as is any other tool that simplify repetitive and boring tasks.

 There is a little, but important catch here. This approach works on the long term, only if the app stays true to its nature, that is, it changes very little and the changes are still about data moved from users to db and back. If the apps changes to model business concepts or flow, you're using the wrong tool for the job.

 This is where DDD comes into play. While CRUD deals with simple data structures, DDD deals with modeling behavior and more exactly, with modeling some real life business aspects into code. It's not about getting some data to be persisted, it's about representing virtually the business flow. The problem that appears very often, is that many devs are using the CRUD mindset and approach. It's like trying to cut wood with a hammer. You can do it, but it isn't pretty and it's very difficult.

 For the majority of apps you know from the beginning what you're trying to build. Do use DDD when you know you're modeling concepts and flows and forget that CRUD even exists. Take your time to understand the domain and don't rush it. It's not about developing fast a quick and dirty app, it's about representing a complex system which will very likely evolve over time.

 Do use CRUD when you're dealing only with data structures needing MINOR processing. But be very careful, because some apps start as an UI for the db and then evolve into modeling concepts and business rules. In probably 99% of the cases you should be able to forsee this, based on what the project owner tells you and it doesn't hurt to ask him about things he wants in the future. You won't get a real answer, but at least, you get some hints about things to come. And apply a good dose of common sense, if someone wants to build a GitHub clone, it's about concepts and processes, it's not a CRUD app, no matter how featureless the app is at version 1.

 In these cases, good architecture is key. Use a DI Container and at least, the 3 tier approach (business logic, ui, persistence). The Repository pattern is also very handy here. Yes, those tiers and those repositories will contain little and trivial code. But the point is, you're laying foundation so that the app can evolve easier into a DDD app. It's a couple of minutes job to setup a DI Container and to come up with a simple wrapper repository. At that point it's unrequired code, but it's a such small price to pay to get the flexibility of evolving the app from CRUD to DDD.

 I repeat though, that proper architecture and careful design is key. You need to achieve a fine balance between the simplistic but efficient CRUD and the complex but much more flexible DDD. There is no recipe for that, it's up to you how you do it. Maybe this process is what separates a good developer from a novice (regardless the 'experience' years). CRUD is simple to learn, a junior dev can apply it very fast and then repeat it for 10 years with minor variations, it's almost like a recipe. DDD is all about your skill, a real challenge to your comprehension, design and architecture skills.

 In conclusion, be honest about the nature of the beast (app). First thing, ask yourself if it's a real CRUD app, before you start churning out code. If it screams business concepts and you need to understand the domain or is all about processes or flows, forget that CRUD exists. Don't make the mistake of approaching such app as CRUD. If it is CRUD, then don't bother with SOLID code (yes, I've said it), use any tool that makes you work easier.


