---
layout: post
title: Repository Pattern and IQueryable
category: Repository
---

Many developers are defining a repository interface with at least a method which returns an IQueryable, especially when they define a generic repository. It's quite tricky to see it, but if you're doing that, you're using the Repository Pattern the wrong way.

 First aspect is that having a method returning a IQueryable is a case of [leaky abstraction](http://stackoverflow.com/questions/3883006/meaning-of-leaky-abstraction). But you can counter that by saying that IQueryable is part of BCL and Linq is part of C# . Fair enough!

 However the mistake is not the IQueryable itself, but its purpose. An IQueryable is codetty much a query builder and it's great for that. A repository method returns some business object (model). Can you spot the difference? You ask the repository to give you a model. How the repository comes up with the model is its problem.

 The point is that using IQueryable, you're asking for a query builder and not for a model. A query builder specifies how the query should be, **how** to get the data. Then why are we using a repository? Isn't the repo's job to know how to get the thing we want? We're using the repository because we want to specify **what** to get, not **how** to get it.

 If you want to have full control over how to get the model, then you don't need a repository. Use IQueryable or some other builder for the purpose. But if you don't care _how_ or _where_ from the model is retrieved, use a repository whose methods return only a model or an enumerable of models. Let the repository do its job of hiding the implementation details. Use it as an abstraction: tell it the what, not the how.


