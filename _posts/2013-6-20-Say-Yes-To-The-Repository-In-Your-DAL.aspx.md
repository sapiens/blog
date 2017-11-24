---
layout: post
title: Say "Yes" To The Repository In Your DAL
category: Repository
---

Once again I read a [post](http://tech.pro/blog/1191/say-no-to-the-repository-pattern-in-your-dal?) about how 'evil' or 'useless' the Repository is. I like especially this part: "Let me emphasize, that the repository pattern is not flawed, or a bad idea in general" BUT " You've given up all the features of your data storage technology, for a bland, neutered, inefficient way of accessing your data".

 This article is a rebuttal for all these kinds of posts. But first let's begin with a nice story.

 In the near (ok, far) future humans set on an exploration mission outside our solar system. But our tech still is not advanced enough to go warp or even speed light so for the duration of the trip the crew must be kept in stasis to survive. We call that device the Stasis Chamber (SC). Meet John, a crew member. He is one of the many who will be 'persisted' in stasis. But what is this stasis after all? Well it's 'something' that keeps you alive for long duration in some kind of sleep. Like criogeny? Maybe... Who cares?!

 What is relevant is that John is sent to the SC, the storage process begins and that's it. When John is needed back, the SC operator press a button to restore John. How does everything works? Nobody cares much. This is how the proper usage of the Repository works. But let's see how the people misusing the Repository would use the SC.

 First of all, let's be clear about it: John is a flesh and blood human while SC uses some crystal cells to store the people send to it. Everyone expects to put a human in the SC and to get back the same human flesh and blood and all. So how would the anti Repository crowd do it?

 Well, they would tell SC to get some energy parts of John from the crystal cells 1 and 2 then join other cells on whatever condition then do something weird called projections (I think) which will turn the energy into John. What's the problem with this approach? Every single operator of the SC MUST know how SC works internally, what types of energy cells it uses how to join cells etc. It's quite expensive to have SC operators this way. And even worse, if you replace some part of SC which uses the more efficient MK2 cells, all the staff has to be trained again. What a waste of energy(pun intended) and credits!

 Why would a SC operator need to be also a SC engineer in order to actually to something? Why not just press a button and be done with it. The actual SC engineer wil know everything about SC internals but the operator (the user, consumer) will just access the behavior it needs without caring about other details.

 When you use the Repository, you're saying: I don't care what, how or where persistence is. I just need my John (sorry, a member with the id 'John') back up and running, because for my context I need John, the domain object and not John the energy cells (the persistence representation of John). I know that those crystals are amazing and do lots of magic but I, the business layer, the domain or any other application part (excluding DAL) I need objects relevant for MY context. The SC (Repository) uses the magic of crystals directly to persist/restore John, but outside the SC nobody really cares about cells, energy and how the magic works.

 Back to our rebuttal. Khalid says that by abstracting EF or other ORM you lose lots of advantages. Well... no. You don't lose them, as the repository implementation still uses them, but the rest of the app won't, as it should be. The fact is, the repository CONTRACT hides the underlying details. Because when you're using the Repository, you only care to get back your object for a specific context, without bothering how. You tell the repo what and not how or where.

 Also a very common mistake is the illusion that the Repository contract deals with storage entities. It does not. It deals with application relevant objects, usually business objects. The User returned by the Repo is the User business object and NOT the User table or document you might have in the db. They have the same name because it makes sense, but they are part of different contexts, layers. Each is a valid model but only in its own context. So when the model is that different (yes, this means you don't have an anemic model) you need a **mediator** between the layers to handle the incompatibilities. In our story, the mediator is the SC, in our code is called Repository. If you're tempted to say ORM, remember that the ORM is the [database](http://www.sapiensworks.com/blog/post/2013/05/15/You-Might-Be-Misusing-The-ORM.aspx) and works ONLY with rdbms.

 Next, Khalid says that one of 'advantages' of Repository is testing and gives an example of how to test EF,RavenDb etc without needing a repository. Great, but you know, the tiny problem of EF/RavenDb returning storage entites and NOT context relevant objects still remains. Of course, it depends, because with a document db you can really store a business object, but it's considered good practice to save a memento and not the business object itself.

 About DRY, well, the examples given remind me a bit of the Repository spirit (irony!) but they're still based on the assumption that the domain object and persistence objects are the same. About the Command approach, I'd keep Commands for communicating between contexts via a service bus. Btw, in the Repository those commands are the Repo's methods. And speaking strictly about Khalid's example: FindPRoductByIdCommand is pretty much a query because it returns something, while a command shouldn't return anything. That is, if you're into that CQ(R)S thinking.

 The issue of storage substitution is great: "Once you've committed over 10,000 records to a database you are unlikely to switch from SQL Server to Oracle or a NoSQL solution.". Hm... tell that to [Twitter](http://www.infoq.com/presentations/Real-Time-Delivery-Twitter) which uses MySql for writes and Redis for reads. Yes, CQRS again. I'm starting to think that people who don't like Repository, don't think their application will ever be big enough to need more than one storage type.

 "Switching data storage is a big decision, the repository pattern makes you believe a switch is trivial. It's as trivial as putting your foot into a bear trap."

 It is a big decision and not trivial, but the Repository helps with the fact you change ONLY parts of DAL, only ONE layer is affected as opposed to hunt and modify code scattered all over the app. With repo I KNOW that no matter what storage it is, the rest of the app will work untouched. This is a GREAT advantage.

 Repository is a tool, like every other design pattern or technique. You use where it make sense. Incidentally it make sense very often especially with domain objects because the way such object is designed doesn't match very often the way it's persisted. When your 'domain' objects are 99% identically to the persistence objects then you don't need it.

 Fact is, you have parts of the app where Repository makes the life easier, parts where Event Sourcing fits like a glove and parts where calling directly the storage gives you great flexibility. To general some view models, it might strictly a simple call to the storage, while in other cases I need to do 4 ugly queries in order to get all the date required for my view model. In the first case, I'd use directly the storage while in the second case, I'd put up a repository. After all, I want a relevant object for the UI context and not just data and certainly I don't want my controllers to be clogged with building queries or calling remote services. In fact, once I step out of DAL, I don't care about database, web service or whatever you're using for persistence. I care ONLY about objects relevant for the context.

 That being said, it's obvious for me why I'm using and continue to use Repository: it makes the code more maintainable. Every time I see someone  saying that you don't need repository when you have ORM or that the Repo is an anti pattern, I say that I don't want ever to touch code written by that person. If she prefers to mix layer responsibilities or corrupting one context's model just to satisfy other context, it's fine with me, BUT I want clean, easy to maintain code. In coding, the real productivity is about how easy is to understand and change things and not how fast you can write lines of code.


