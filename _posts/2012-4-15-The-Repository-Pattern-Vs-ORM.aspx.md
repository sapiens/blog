---
layout: post
title: The Repository Pattern  Vs ORM
category: Repository
---

It happens that every once in a while I stumbled upon this question: "Why use the Repository pattern when I'm already using an ORM?Isn't the repository redundant in this case?". Well, as always you have people arguing that the Repository is an [anti pattern](http://ayende.com/blog/3955/repository-is-the-new-singleton) and the other who says otherwise. But who's right? Is it the repository useless when you have an orm? Let's find out.

 An ORM is the acronym for Object-Relational Mapper and this term is a giveaway. It says clearly that it's purpose is to provide a bridge between the object oriented way(your application) and the relational way (the relational database). Pretty much any ORM will provide the application with a virtual oop representation of the database and will make it easy to CRUD without involving SQL that every developer loves. It also provides an abstraction over a specific rdbms, so it's easy to change let's say from MySQl to SqlServer with a simple config setting. However, if you want to use a NoSql db like MongoDb or RavenDb, you're out of luck. These are document databases not relational ones. An ORM is completely  useless here.

 But with a RDBMS, it's very useful and it can simplify your work as a developer, especially if you don't like dealing with sql and specific db quirks. However it does come with a price: steep learning curve (it's not that easy to understand how to fully use NhIbernate or EF for example) and some performance penalties. It's also way to easy to intermingle it everywhere, forgetting that is in fact an implementation detail of how the app is persisting things(the fallacy of [using the ORM model as the domain model](http://www.sapiensworks.com/blog/post/2012/04/07/Just-Stop-It!-The-Domain-Model-Is-Not-The-Persistence-Model.aspx)).

 And that's where we enter the Repository domain (sic). The Repository Pattern is an architectural pattern which abstracts and encapsulates anything persistence related. Now, the persistence medium can be a relational db, a document db, an in-memory db, a simple file, a cloud storage etc. It doesn't matter, as the Repository hides that. The application doesn't need to know all these details. The app doesn't care you're using MySql or RavenDb, it just cares that the persistence can store and restore objects.

 The Repository pattern, allows the application to know only about abstractions. As far as the app is concerned, the repository is the place where objects are stored and retrieved. The Persistence Layer pretty much contains just implementations of specific repositories used by the app. And the repository implementation actually does the job. How it does it, it's an internal concern, an implementation detail of the Persistence Layer, so the app doesn't know nor care about it.

 Of course, if you're using a RDBMS (as it's very common) you might use an ORM. Maybe it's easier for you to write the queries, maybe you want to change later to a different RDBMS. That's no problem, use the ORM, the Repository won't mind, but the fact that you're using a rdbms and an ORM is still an implementation detail, known only by the Persistence Layer. Further more, different repositories might use different storage systems. I can store most of the data in MySQl but I'm still storing images in the file system and the app only knows about a IFileRepository interface .  The actual FileRepository object may use the db to retrieve metadata about the files and the file system to return the files.

 Thing is, within a repository implementation I can use ORM, rdbms, cloud storage, file system etc. I can change the implementation any time, the app won't care and it won't be affected by the changes.  That's why using the Repository Pattern makes it much easier to do TDD: you can mock it or stub it very easy.

 In conclusion, an ORM and the Repository Pattern have different purposes so it's not a matter of X vs Y. Use the **Repository** because you want to abstract and encapsulate **everything storage related** and use an **ORM** to abstract access to any (supported) **relational database**. The Repository is an architectural pattern, the ORM is an implementation detail of the Repository.


