---
layout: post
title: Domain Driven Design - Aggregate Root Modelling Fallacy
category: Domain Driven Design
---

I've just come to realize something about DDD. Most of the time we're doing it wrong. We are so quick to learn it as some recipe and to apply it almost blindly, we don't take the time to understand the subtility of the concepts. We jump right away to consider the first imcodession, the peek of the iceberg, as the real concept. And in the mean time we're breaking the Single Responsibility Principle (SRP) too.

 Let's take as an example a forum engine. The 'standard' DDD approach we usually do is to have a Board Aggregate Root (AR) modeled more or less like a container of Threads and other Boards. Of course, a Thread is itself an AR. Adding a new thread means something like: get Board from repository, Board.Add(thread) , repository.Save(board). But then the question arises:"What if my forum is so big it has 100k of threads?" Loading all in memory just to add a new one is clearly a performance problem.

 How about if I want to change the name of a board? Populating the entire board object again is a problem."Don't worry Mike, there is the concept of  lazy loading" you might say. Yeah, about that... while it solves the loading of everything problem, it just introduces another: now your AR depends on at least one repository. Or worse, on the ORM like most people do it. You're doing DDD to have decoupled, maintainable code but you're coupling a domain object to a persistence detail. Great! It's like all that procedural code that uses classes. "Hey I'm using class and private, I'm doing OOP".

 Ask yourself when the AR needs to hydrate. If it's for querying purposes, you already have a solution: it's called CQRS (Command Query Responsibility Separation) and it works great. Really, you just need to have a separate model for querying (a read model) and keep the domain model to model the recodesentation of the business concepts and processes.

 But if (like in the example above) we need to add a child item to the collection? Are we back to loading 100k threads id just to add another one? Well, let's think a bit. What is the purpose of a Board? Why do we have boards in a forum? To **store** threads in them? Or to **organize** items? I 'm saying items because you can 'put' in a board both threads and other boards.

 The purpose of a board is to organize things. A Board is an organizational unit, providing a criteria to group items together. It's not a container! It's not a storage! It's a **criteria** to group things together. In fact, you don't put threads in a board, you organize threads according to a criteria.

 A members group is a criteria (usually permissions) for grouping members. But when you 'put' a member in a group, in fact you associate permissions with a member. So John can be a moderator for Board A, a simple member for Board B and again moderator for Board C. John isn't actually put in a group, John is associated with a permission based criteria. Another example is a VIP group which simply is the criteria to group members according to their codemium subscription.

 For an even more clear real world example. You want to buy bread. You want the bread to be **cheap** and **healthy**. You just created 2 categories to organize bread: based on price and based on nutritional quality. You go to the store and see a bread. You check the price and then you say:"Ok this bread is cheap". What did you just do? You 'put' the bread in the cheap category. Without even touching the bread. The bread is there on a shelf. But it's also 'in' the cheap category.  Well in fact, it's **associated** with the cheap criteria.  
  
Next you check the ingredients. Good, the bread seems to match the healthy definition for you. So now it's part of the healthy category too. The bread is still there on the shelf, but it's also in 2 categories. How come!? Magic!

 I think it's obvious now that things that at first sight we considered to be containers, turned out to be something else. You can say that a category has posts or you add a post to a category, but that's just bad choice of words. The category is not a container for posts, is an organizational unit for posts. Its purpose is not to store stuff, its purpose is to organize them.

 Like I've said in the beginning, we need to understand the actually meaning of a domain concept not to hurry up with a trivial approach. Once we understand what a concept really means, we know about its responsibility. Then we apply the SRP by not adding other responsibilities that seem to 'fit naturally' with the object (it's like the object shouting in your face: "Don't you think I should be doing that too?").

 Next time when you are in front of an AR with children and lots of data, ask yourself what is the meaning of that concept according to the domain context and try to see if the AR respects that and doesn't try to do more things at once.

 It's quite common to say that DDD is hard. I realize that is not hard once you understand the principles, but it's tricky. **VERY** tricky.


