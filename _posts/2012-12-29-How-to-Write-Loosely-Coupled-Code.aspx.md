---
layout: post
title: How to Write Loosely Coupled Code
category: Best Practices
---

Any developer who want to write good code knows that good code is loosely coupled by highly cohesive. But look around and see how many apps really are built that way. Not so many. Why is that? I mean, the 'loosely coupled, highly cohesive code' is almost a mantra repeated by anyone. Is it really that hard?!

 So when two or more objects have low coupling? When an object doesn't depend directly on another object. That is, it doesn't depend on a concrete implementation of some behavior. Instead of using directly a _Database_ object, how about using an abstraction, like _IPersistState_? The Database object can implement that interface, but the consumer isn't coupled to that concrete type. It's coupled to the behaviour, to a feature. And it doesn't care which concrete type implements that feature.

 What about the highly cohesive part? Highly cohesive means that related code should work together. It's a bit tricky to understand it at first, but it's very easy with this example: A hammer and a nail are highly cohesive (real life) objects. They work together properly. A rock and some nails are highly cohesive.

 They are also loosely coupled. Why is that? Because you can use any type of hammer (or rock) to put those nails in their place. And those nails can be long, short, thin, think etc.

 The trick with loosely coupled, highly cohesive code is to use only abstractions. You don't really need a hammer for those nails, you need any object that can be used as a hammer. Transposing into code, this means that an Worker object will take an IHitNails dependency, which can be implemented by a Hammer or a Rock type.

 See? It's not hard at all (j/k).

 I think that not many developers are writing loosely coupled code, because of 2 things:

  
  2. They don't actually understand the benefits; 
  4. They lack the discipline to not write the first thing that crossed their mind. They simply rush things over, instead of taking the time to actually think before they write the code.  So let's talk about the benefits.

  
  * As the coupling is against an abstraction (the what, not the how), this means the actual concrete dependency can vary according to the context. The consumer code doesn't need to change at all. And you can also change how a concrete type works, without worring that you might break something. Very useful! 
  * Testing is much easier. While some devs think of testing as more of an exotic practice or as being the same thing as TDD, testing is an important tool to simplify your work. Having unit tests, doesn't mean you're doing TDD. TDD just means you write the test before the things you want to test, that's it. With testing, you can focus on a specific parts of the code and you can verify that it works properly, without requiring to build the rest of the application as well. So if John work on the database stuff and you need a repository, you don't have to wait for John to finish his work. Taking an interface of the repository as a dependency means you can mock it or use a fake implementation for the testing part. And after the test passes, you can move on to other stuff, knowing that the code works as intended. 
  * Having code immune to side effects and tests that can catch faulty behavior means that you'll spend less time debugging and changing things and new features can be added more easily. On the long run you'll save time. That's improved productivity!  About the dicipline, it's a matter of self-control. If you like rushing away to code stuff, you'll lose more time on the long run as you have more code to change when you realise that the first instinct wasn't the best one. Just try to understand things and how they fit into the big picture before you start typing code.

 Loosely coupled code means to apply SRP (Single Responsibility Principle) and to avoid using concrete types for things you feel they might change. Even better, try as much as possible to make your code depend on behaviors of objects (this means interfaces) instead of the objects themselves.


