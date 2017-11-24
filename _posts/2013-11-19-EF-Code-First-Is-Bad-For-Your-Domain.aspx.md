---
layout: post
title: EF Code First Is Bad For Your Domain
category: Domain Driven Design
---

We already know to not [use the ORM entities to model the Domain](http://www.sapiensworks.com/blog/post/2012/04/20/Dont-Use-ORM-Entities-To-Model-The-Domain.aspx) but what about the magic feature of Entity Framework (EF): Code First? Many people say it allows you to write your domain objects FIRST and then use EF to map them to tables.

 Let's remember what the ORM stands for: it solves the mismatch between RELATIONAL TABLES and OBJECTS. No, it's not about mapping Domain Objects to tables, it's about faking OOP on top of tables. And this means that the relational mindset drives the design of those objects, better known as ORM Entities.

 Let's get back to EF Code First (EFCF). I've created my Domain entity

  
```csharp
public class User
 {
   public Guid Id {get; private set;}
   
   public LoginName Name {get; private set;}
   
   public PasswordHash Password {get; private set;}
   
   public Email Email {get; private set;}
   
   // other properties, constructor etc
   
   
   public IFormattedContent SomeContent {get;private set;}
   
   List<Trait> _traits=new List<Trait>();
   public IEnumerable<Trait> Traits
   {
    get{
           return _traits;
     }
   }
   
   public void Add(Trait t)
   {
     //some rules
      
     _traits.Add(t);
      
   }
 }
```
  See something interesting here? The majority of properties are value objects. This ensures the Entity is always in valid state and you can't have "blsdfds12312" as an email. I also have some "children" a collection of traits (invented for this example). These traits are required to define the concept of User, they are not just associated with an User (there is no 'has' relationship).

 Most of the examples you see with EFCF look a bit different. The entity has very rarely value objects and if it's the case, in 99% (read: always) it acts as glorified container for other objects, the children. And as magic happens it's an obvious one-to-many or many-to-many relationship between the 'parent' and the 'children'. It's like straight out of the DDD blue book, right?

 No, this is the good old relational mindset here, which has nothing to do with DDD. EFCF works so well with CRUD apps. I couldn't find an example of using EFCF with a business object. Everything is about data structures having relations with other data structures. It seems that everyone defines tables using objects. People are using EFCF to define the db structure! Not much domain here. In fact, so much of the 'domain' objects out there are data structures named entity, aggregate root etc. Their properties represent relationships and not bits required by the domain concept. It's just bad modelling who just happens to resemble a lot the CRUD mindset.

 Back to our domain entity. How would you map the above example? I simple don't know how to do it. Will EF create a table for each value object and then use joins for every query? Perhaps it will flatten the value object to one value per property. I could write the mapping convention myself but this is like doing manual mapping.

 How about the SomeContent property? How to store an abstraction? And let's talk about Traits also, they should be stored inside the entity as they are part of the definition.

 Suddenly, if you're working with real domain concepts and you do proper modelling, you have a lot of mapping to do which isn't straightforward and EF won't do it automagically for you. You need to tell it how to do stuff and at that point it's like you're doing the mapping manually. And the queries will be fun too. And chances are, you'll end up changing your Domain model to be more compatible with EF, because it's easier that way.

 Mixing DDD with magic tools isn't straight forward at all, it's tricky and like a siren call, by the time you realize where you are, it's already too late (and too expensive to change something).

 If you want to keep things clean and your life easy, first understand that CQRS is a must with DDD. This means that the ugly Domain entity can be easily put in a doc db or just serialized somewhere. Very easy, no mapping required. The ORM has no role in here. Then create a read (queryiable) model that the ORM will love and it will work without hiccups.

 It's annoying that we have so many silver bullets around: ORMs, SCRUM, DDD, TDD etc when they are only tools or mindsets which need to be properly understood then properly used. I know the human brain always searches for shortcuts but in this case, the misusing of the ORM with DDD leads to a swamp from where you won't come back.


