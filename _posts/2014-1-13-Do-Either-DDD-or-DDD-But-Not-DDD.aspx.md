---
layout: post
title: Do Either DDD or DDD But Not DDD
category: Best Practices
---

If you think this is a joke, well, it isn't. Usually there are 3 ways to design an application.

 Domain Driven Design (DDD) is best fit for rich business domain. The Domain is the most important part of the app, the core around which the rest of the app is built. You start with the modelling of the business concepts and use cases. The design is driven by the business.

 For CRUD apps, you can employ Data Driven Design (DDD). You define the data structures which are the app's core. All that matters is the data that we want to input, store and retrieve as fast a possible. Everything is centered around the data structures.

 And there you have one of the most used approaches, (Relational) Database Driven Design (DDD). You start the app by designing the database schema, tables and relations. Everything revolves around the database and relational rules. You have only one problem though: it's kinda the wrong approach.

 In the first 2 DDD approaches you have either the rich behaviour or the data structures at the centre. They are the reason the application exists, the value the software brings. But the db driven design is all about the db. Is the reason of your app, the database? Is it its purpose just to store things? If you want that, every major RDBMS vendor has some kind of an app builder, that sits on top the db and allows you to enter data and to create queries (reports). Why bother wasting time and money developing a whole new application?

 Starting with the db design is harmful for both your rich domain or the data structures. Because you will employ a db centric mindset and put the needs of the db before the reason for which the app exists. Even when dealing with data structures, you want those structures to be defined according to the business needs and not according to relational mindset. You shouldn't care about one to many, indexes or normalization when designing, because your app isn't about those.

 Database cares about one things only: persistence of data. Yes, you want fast queries, but that's an implementation detail. All the persistence details should stop at the Persistence Layer boundary. They shouldn't exist outside Persistence. Rich or anaemic behaviour, it doesn't matter. The application's core design should be ignorant of the persistence.

 Every time you start your Domain Driven Design or Data Driven Design with the database, you're just making your life harder. Your 'domain' objects or data structures will be designed to serve the rdbms first and the business second. This is NOT what you want.

 Design the business objects first, the business model. Use the repository to abstract persistence i.e ignore the database (this means use in-memory repository implementations). After everything works properly, go to the database and design the schema best for what the app needs. If your persistence model doesn't match 1:1 the business model (anaemic as it might be), just use a mapper. Keep your models clean and focused, serving only one master (context).

 In conclusion: do either DDD (domain driven design) or DDD (data driven design) but never DDD (database driven design). 


