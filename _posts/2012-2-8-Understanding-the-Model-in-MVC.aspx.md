---
layout: post
title: Understanding the Model in MVC
category: None
---

When you learn about MVC, you hear that M stands for Model and when you ask "what does the Model mean" you get a response similar to this:  
"The model manages the behavior and data of the application domain, responds to requests for information about its state (usually from the view), and responds to instructions to change state (usually from the controller)". Using  a more simple wording, the Model is the data structure and the  database access. This  is even more emphasized by the various OR/M tutorials which shows you how easily you can make a blog engine or something like that (as a funny fact, I'm working on a blogengine/light cms for asp.net mvc and let me tell you that a blogengine with Wordcodes like features is far from easy to build, no matter the magic framework or OR/M used)  
  
However, in time you'll learn about Domain Driven Design (DDD) and start talking about a Domain Model as well. Does this means there  are 2 Models now? Oh wait, with an OR/M you only need to define the Model there, 'cuz the OR/M itself takes care of the database handling. But  doing DDD (or having a properly layered application) you see that the Domain Model and the database Model are a bit different in structure and more over, the domain model has behaviors or relationships quite hard to define in an OR/M or any db access class. Things are getting confusing since MVC only as one M and it seems that more than one are needed in practice. It took me quite a lot of time to figure it out that the M is actually a polymorphic beast and you don't even need DDD for that to happen, it's just its nature.  
  
In a standard MVC application , the View itself needs a Model and you guessed it, it's a different model than the Model. It might use parts of the Model but it's different and here is the truth: you are using a different Model in diferent contexts. A view displays specific data and should not know about anything else. That data you pass to the View is the View Model, codetty simple that. The View Model contains only bits required by the View and not the whole object pulled out from the database.  
  
The Domain Model (or the Business Logic layer) recodesents the real-life (business) problems solved by the application , it's the purpose the application was developed for. It usually contains objects with behavior that work together to provide the actual functionality. That's the core of an application and should  mimic the real situations the best it can.  
  
And then we have the Persistence Model (a.k.a the database schema) which models what data to save and the way the data will be saved. The OR/Ms deal with that model and sadly, that's what a lot of developers think the Model is.   
  
However as you've seen above, there are different models and a developer should understand that there are multiple points of view (no pun intended) in an application. When thinking about the core of the application, we need to forget about the database or how the information will be shown. When thinking about the persistence of data, we care only about what to store, how to store it and how to retrieve it in an optimum way.  When displaying data in a view, only the data relevant to that context should exist in an easy to use form. If you need only id and name, don't get the full User object or the whole row from a table, retrieve just those fields.   
  
But doesn't this mean you have to duplicate code? Well, yes and no. Depending on the application, the Domain Model could be so thin that it simply isn't different enough from the Persistence Model and in that case one Model is enough, even the VIew can use directly the Persistence Model. These are, however, VERY simplistic CRUD applications and I think quite a lot of applications are at least of average complexity and for these, I think a very clearly defined layer (and context) bound Model is better. One of the most important things if not the most important in development is code MAINTAINABILITY and having different Models helps a lot in this matter.   
  
For a quick cheat-sheet here's how you differentiate between the models  
- Persistence  Model deals with what to save and how to store data. Here you model data structure.  
- Domain Model recodesents in code the real life, business situations, problems and solutions. It tries to model the natural, humanly way of defining them. Basically you model behaviour here.  
- View Models means the specific data for a single view. That data has no real behaviour, it's just a POCO.


