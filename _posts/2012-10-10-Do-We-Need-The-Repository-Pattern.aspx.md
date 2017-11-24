---
layout: post
title: Do We Need The Repository Pattern?
category: Repository
---

Just when I thought I've covered anything related to the Repository Pattern, Jimmy Boggard writes [this](http://lostechies.com/jimmybogard/2012/09/20/limiting-your-abstractions/) and [this](http://lostechies.com/jimmybogard/2012/10/08/favor-query-objects-over-repositories) . Since I consider the Repository Pattern the easy and intuitive way of dealing with storage concerns, I'm always baffled how other devs seem to consider it a complication or not needed. I'm really starting to ask myself if I've understood the pattern correctly. But even if my view of the pattern is -let's say - extended and maybe it's actually another pattern or a combination, I still find it the most natural way of abstracting the persistence.

 One thing I keep seeing popping up is the if you're using an ORM what's the point of the repository? You can read my post[ Repository vs ORM](http://www.sapiensworks.com/blog/post/2012/04/15/The-Repository-Pattern-Vs-ORM.aspx) for more details, but in short, the ORM only 'hides' a handful of RDBMS. Change the RDMBS to a NoSql and the ORM becomes useless. Yes, you might not change the db very often but the point of a repository is not to hide a db engine, it's to abstract the whole storage.

 In an application, you might deal with different storage at once: local db, xml files, cloud storage etc. The Repository pattern hides all those and gives to the rest of the app an interface: get me this, save that. Where and how is not the app's problem.

 Let's work a bit on Jimmy's example. We have the simple case of a blog engine and a controller action which returns a list of posts. But querying the RavenDb in the controller is ugly (hard to maintain - fat controller symptom) and the elegant solution is to use a query object which encapsulates the actually query, pretty much a facade.

 For me this approach is actually a bit of the repository pattern, because that why you're using a repository in the first place. Yes, to abstract anything storage related **and** to provide a facade that makes sense from the business point of view. A 'GetPosts' method says more that a 5 lines long Linq. It helps with maintainability.

 But about those repositories with dozens of methods? Well, split them. That's why you're using interfaces, are you telling me that you'll use all those methods in one controller? If so, then that repository is actually needed for that controller. But if the controller only uses 2-3 methods, extract an interface and use only that.

 Furthermore, a repository usually serves a bounded context. The **PostsRepository** from the Domain is very different than the **PostsRepository** from the UI (different bounded contexts). The latter is usually named something like **PostsQueriesRepository** **PostsViewModelsRepository**(in fact it will be an **IQueryPosts** interface) to avoid confusion.

 I was asked a few times, what's the difference between a repository and a DAL interface. The repository is a high level concept  used to access the persistence. It's implemented in code first as an interface (most of the time) defined where(in any layer) you need it. The actual repositories implementations reside in the DAL (Persistence Layer). And you can say that a repository contract is an entry/exit point of DAL, while its implementation is a a part of DAL. So the DAL is made up from **ALL** the repositories and **ALL** of DAOs, ORMs, ORM entities and utilities, pretty much everything directly related to accessing the storage.  
  
It seems that most of the devs are considering a Repository just a way of hiding a db or worse, a wrapper over the ORM. It's a bit more than that, it's a high level abstraction which helps separate the Persistence Layer from the rest of the app. It also provides a meaningful facade from the business point of view, thus helping with maintainability.


