---
layout: post
title: Tips And Tricks For The Repository Pattern
category: Repository
---

**1**. Remember that the main purpose of the Repository Pattern is to decouple the application from the persistence. It's not about abstracting sql or changing databases, it's all about decoupling.

 **2**. Design the repository interface on the way, as needed. If you have MyDomainEntity don't automatically include a IMyDomainRepository with CRUD methods. Write the interface ONLY when you'll be needing it and don't try to come up with all the methods you think you might need. While most of the time you'll end up with CRUD methods (especially when dealing with domain repositories), define those method only when there's an actually use of them.

 Even more, think about the expectations the context when calling a certain method. Here's an example:

  Both methods, have in their comments the exceptions the app expects to be thrown when certain conditions happen. Note that at this point, there isn't an actual repository implementation.

 **3.** It's common when defining domain repositories to only have the CRUD + Get methods. This almost begs to use a generic repository interface. But here's the catch: That generic type MUST be a Domain object and NEVER a Persistence object. This is where all the misusing of the Repository pattern happens. Ever more, it's the generic repository INTERFACE we're talking about not a generic implementation. The generic implementation works only when you just serialize everything i.e the repository doesn't really care about the domain object is dealing with.

 The CRUD+Get repository definition is suitable only for Domain/Business layer. For querying purposes, you'll have a Query/ViewModels repository (you can safely call it whatever you like even service, but it's still the repository principle applied) which has only Get methods and returns ViewModels/DTO. The implementation will query the db, create and populate the view models. Remember that one of the benefits of the Repository is removing the 'noise'. Your app should only see "repo.GetOrders(new MyCriteria())" and not be bothered with constructing the whole query. The repository keeps things clean and more importantly semantic. It's about the WHAT not the HOW.

 **4.** Implementing the repositories, should be among the last things you do in an app. I know that it's strange for many people, but this is the proper way to build a solid, decoupled app. Of course it gives the benefit of not caring at all about what db you'll be using and how things will be stored. It allows you to design the domain objects properly, instead of asking you at every step 'how would I store that? Can the serializer access that private field?'. You can delay that decision until the end. Which is great!

 So how can you test or demo the app? Well, you have to implement the repositories but you'll be using an in-memory collection (a list, array, dictionary etc). Why that, instead of the real repository? Because:  
    a. It's the fastest implementation that can be written.  
    b. You'll be doing it using TDD, so it's pretty much a pretext do write the repository tests, because I hope you like having tests for the repositories too, right?

 Of course, those tests will be waiting for you when you'll write the 'real' (any) implementation. Here's an example:

  All the repository tests are contained in one abstract class. All tests are using the abstraction, so the tests simply don't have a clue about the actual implementation. Here we're testing 2 implementations: the LocalMembersMemoryRepository and the LocalMembersRepository, which is the 'real' database enabled implementation. Each gets its own testing class which just setups and tears down the fixture. Adding a new repository implementation (say for RavenDb) means just to code it, then add a new test class which inherits BaseLocalMembershipTests with code for init/destroy storage. Run the tests and that's all. Once everything is green, I know the new repository is solid.

 This approach gives you the following benefits:  
    a. You're building the easiest implementation the right way (TDD) and you have something the app can work with, without messing around with a real database.  
    b. You can code ALL the application without touching the database and thus, without being influenced by it. The advantage here is that you let the app tell you all its needs for the persistence. When you get to play with the database, you already have all the use cases and tests available, so you'll have a better understanding on how to design things at the db level. After all, the app only defines abstractions (interfaces) and one repository can implement multiple interfaces if that's the most efficient way. At that point, implementing the repository is just a formality, you can let a junior do it, you have the tests in place to ensure the repository will behave as it should.

 **5.** One piece of advice when coding in-memory repositories. Since that 'storage' needs to be persistent, you might be tempted to use static fields. Stay away from everything static. Code that class normally, but do use locking on collections to be thread safe. Then configure the DI Container to treat that instance as a singleton. You'll get the static benefit without any of the drawbacks.


