---
layout: post
title: How I've quit using the Singleton Pattern
category: Best Practices
---

First, let's notice the difference: a singleton is a single instance of a class while Singleton is the name of the design pattern which provides a way to ensure a singleton. Now, that I've cleared this up, I'm using singletons very often but not the Singleton.  And you know how I did it? Without even trying.  
  
Initially I wanted to write a post about when it's appropriate to use Singleton, you know the 'Singleton is evil' story. Some people are not that adamant about it and say that abusing and misusing the pattern makes the Singleton evil. I was one of those people so I wanted to find an example when the pattern can be used without any problem. But looking at my code I've noticed that I haven't use Singleton for a while and it's not because I avoided it. I just didn't need it.  
  
For most of my applications I'm using a dependency injection (DI) container and whenever I need a single instance, I configure the container to serve one instance only. So no need for Singleton there. But what if you don't know/don't want to use a DI container. If you don't know how, it's very useful to learn because you will need it sometime in the future. But if you don't want to, what can you do?  
  
Well, I understand it's a bit overkill to setup a DI container only to manage some singletons, but is the Singleton the solution in those cases? Let's see, the reasons because Singleton is considered evil or an anti pattern (that is a pattern commonly used but with ineffective or even worse results) usually are:  
- Breaks the SRP (Single Responsibility Principle) as you have a class managing its own lifecycle .  
- Hides dependencies, you won't know that an object needs the Singleton until using the object itself.  This is the same problem when relying to much on static methods or properties of an external object.  
- Difficult to test, because a Singleton keeps that state for the entire application (in this case testing), breaking the testing isolation requirement (each test should run independent of other tests).  
- It's basically a  global variable, which has no place in a OOP world.  
  
So we have all this reasons for which you don't want to use a Singleton, but once again, what to do if you need a single instance of a class? I've recently coded a small application for personal use. I coded it as fast as I could, didn't care about good OOP or design patterns etc. I still used the Repository pattern because I like to keep separated the persistence logic from the rest of the logic, but that was it. I didn't used a DI container but guess what? I didn't need singletons. And even if I would need one, I'd just create the instance once and store it in a static property of the application.  
  
Truth be told, if you want to do good OOP, any dependency of an object should be either injected via a constructor or passed as a parameter to a method and you simply wouldn't need the Singleton. If you just want to put together fast an application without regard to code quality and maintenance then it doesn't matter if you use the Singleton or any other (anti)pattern. In a way the choice is dictated by the context and by your intentions.


