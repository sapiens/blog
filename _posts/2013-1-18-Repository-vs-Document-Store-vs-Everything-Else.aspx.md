---
layout: post
title: Repository vs Everything Else
category: Repository
---

Every once in a while someone tells me that the Repository is an anti-pattern because Ayende said so. Go see a demo of Entity Framework and voila, it does all the magic so you don't need a repository (of course, it assumes that EF entities are the domain entities but who cares about such trivial details, we're doing just CRUD apps anwyay).

 I'll keep using this pattern for the following reasons (all together):

  
  2. Any ORM or Document store or NoSql provider are closely coupled to a specific storage type. It can be RDBMS, it can be Key Value data store or document store etc, the point is any of those solutions serve a specific storage. 
  4. A repository hides the queries or whatever is required to work with a specific storage. The point is to tell the repository **what** to get, not **how** to get it.   
     Once your repository returns an IQueryable, you're doing it wrong. The repository knows about business semantics and **GetTopSellingProduct** says more than a complicated Linq or Sql would do. It doesn't matter that **TopSellingProduct** is actually a view model, it does matter that the name tells something meaningful. Oh wait, it's not a Repository, it's DAO! Oh wait, it returns objects that have meaning for a higher layer. "Yes, but they are simple DTOs, they have no behavior!!!" Call it whatever you like, I find it annoying to say Repository **only** when I'm dealing with business objects and DAO when dealing with view models when the idea is the same. Let DAO be used when dealing with local CRUD applications and Repository when your application is designed using business semantics and doesn't care about how and where the things are persisted (after all, you can work with in memory, cached data). You know, when I'm asking the Repository for some view model, I consider the result a model for my view (it isn't just some data). And I don't really care how the storage persists it. I don't even care about **where** the repository takes the data from. Today is the local db, tomorrow is that web service. Bind your code to something from an ORM (or insert here your favourite data access object :) ) and you have more things to change.  Just KISS and use the Repository. It might upset Ayende or others but it will be easier for you to have maintainable code.


