---
layout: post
title: The Repository Pattern For Dummies
category: Repository
---

You read a lot of tutorials about the Repository pattern which seem to contradict themselves. You asked the question on StackOverflow and you also got conflicting answers. What can you do? Can anyone explain this pattern in a _simple_ manner without throwing all kinds of code and codetentious buzzwords at you? Yes, I can!

 Hello, I'm the Business Layer. I hold everything related to the application's Domain. I'm using a lot of business objects and I do create a lot of new objects, too. I know that all these objects need to be persisted or retrieved from some storage. So, I have these shelves where I put all the new objects on. And everytime I need to work with an object I just get it from the shelf, do the work then put it back. I don't know how these shelves work and how they're actually storing or retrieving **my** objects. I do know that I just put the _business objects_ there and I get them back _exactly_ how they were. It's like magic or something...

 Technical people (aka people who like to complicate things) are calling these shelves "Repositories" . As a Business Layer, I have no clue how these are implemented. I only know about some interface of a sort, like a user panel, they call it an abstraction, which accepts my new Business Objects and returns saved Business Objects. I really like this magical shelf, because I don't have to do any work myself. I just ask it to "Get object with Id 2". Or get me all the objects matching a criteria. And it just works!!!

 For you technical people, it looks like this (C# code)

  

```csharp
public interface IMyRepository
 {
	BusinessObject Get(Guid id);
	void Save(BusinessObject item);
	IEnumerable<BusinessObject> Find(MyCriteria criteria);
 }

```
  Since I know ONLY about my business objects, my magic shelf (aka repository) will work only with those objects. And you can see that, amazingly enough, the interface-like panel is actually called an "interface" at least in some programming languages. But I don't know anything else about my magic shelf. Only that interface. And it's the only thing I need anyway.

 Now let me tell you a bit about that criteria stuff. You see, in many use cases I do need to use more than one business object, I actually need to use all the objects which for example have the LastUpdate older than an year. So, I'm telling the shelf to get me all objects matching that criteria. In technical terms, I'm creating a criteria object (which make sense for me, the Business Layer) which I'll pass to the Repository. The Repository takes that object and then applies some black magic on it (I really don't know what the Repository does, since it's **not MY concern**) and voila... it returns me exactly the objects I've asked for.

  

```csharp
public class OlderThan
 {
	public DateTime LastUpdate;
 }

```
  Of course, for this simple scenario, it's easier to just have a special method like this

  

```csharp
public interface IMyRepository
 {
	/* other methods */
	IEnumerable<BusinessObject> GetOlderThan(DateTime lastUpdate);
 }

```
  I'll keep the criteria class for scenarios where I need to pass more than one parameter, but it's not like there's a rule, it depends on what's more easier or semantically suited for that use case. It's also important to say that I decide the shelf's functionality. In tech speak, this means that the _repository interface is designed by the business layer's needs_. That's why all the repository interfaces reside in the business layer, while their concrete implementation is part of the Persistence Layer (DAL).

 Oh, by the way, these shelves are **ALL MINE**! Only **I** can use them. I don't care about UI, Presentation, Reporting etc these shelves are only serving **me** (myself and I). If UI wants to do some queries, let it define its own shelves. I won't share mine. I've created them ONLY for MY purposes and they have _ONLY_ the functionality **I** need! I say that everyone should have their own shelves. Sharing is not caring, sharing is messing (around).

 A special mention to DDD. When I'm called the Domain (ehe!) my shelves are not working with ANY object, but only with Aggregate Roots (AR). And since these AR are modelling important (sometimes complex) concepts, it's a tremendous help that my shelves can handle them so easily, I mean I can only imagine the work they do in order to retrieve such "interesting" objects. As for my 'work', nothing changes, I still tell the shelf to get me this or that. Business (ha! a pun) as usual.


