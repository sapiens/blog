---
layout: post
title: DDD Tutorial - Modelling Query Cases 
category: Domain Map Tutorials
---

This tutorial is an excerpt of the _actual_ DDD modelling process of [Value++](http://www.getvpp.com/) (Agile project management software) using [Domain Map](http://www.domain-map.rocks/).         

# Business functionality: We want to list the members of an organization

In order to see or manage the members of an organization we need to get a list of them. This is a very simple and straight-forward query business case. We only need the following data: member id, member name, member role.  
 
# Modelling

## The bounded context

The font end of **Value++** is a single page app (SPA) that communicates with an API server. The backend app doesn't have a `UI` layer/component, but it does have an `API` one. Every query functionality invoked directly by the client will be implemented as part of this component. So, in Domain Map, we need a `API` context that needs to be created (click on the "+" icon below the selected bounded context button).

## CQS in action

It was obvious from the start that we're dealing with a Query, no surprises here. But what does it mean from a modelling point of view? Well, it means no changes (events) or aggregates are required. All we really need is some read-only data i.e a **read model**. But this is still an **application** model, not a _persistence_ model, which is an implementation detail.

All the read models of the domain are about _what_ data is required by a specific functionality and they stay the same, regardless of how you choose to implement persistence. Actually, the more query cases you're modelling, the better you get an idea of how the persistence should be designed. In DDD and Domain Map it's all about information.

But first, let's create our Query case: Add -> Query Use Case and enter the name: "Get organization members". After that, click on the `Query Use Cases` tab, then click on the newly created item to see the details.    

Now, create our read model: Add -> Read Model then enter the name: "Organization member". Once it's created, click on the "+" icon from the `Involves box` then select our read model. This tells us that the query case uses the read model.

![query links read model](http://i.imgur.com/m79Zoh7.png)

Click on "Organization member" link to go to its details. Here, we just write the structure of our read model.  

![read model](http://i.imgur.com/BWTIarf.png)

One thing I like when dealing with read models is that they are just simple data structures and quite literally we're actually designing the class structure that we'll use in our code. A _lot_ of Query cases and read models are just CRUD bits so it's best to treat them as such. But we don't care much about implementation details when we're modelling so, for technical details, it's best to create a note. In this case, I don't have anything useful to say so, I leave things as they are.

And that's all you need to model a very simple query business case. In some case more than one read model can be involved, so add as many as required, but keep in mind they still need to represent semantics for the business.

# Bonus case: Get all organizations a user is member of

Very similar to the one above, we need to return a list of "Organization", where the needed data is: org id, org name, member role.

We're already in the `Api` context, no need to change it. Add -> Query Use Case and enter: "Get user organizations". Then go to the query use cases tab (if you're not there already), select the new case then Add -> Read Model and enter: "Organization info". This name might change in the future, but for now it's good enough. Link the read model to the query case, using the `Involve` box. Go to the read model details and write its structure.

So far, it's identical to the previous case (to be honest it gets boring after a while, but hey, it's the functionality the business needs and it's simple enough). But now, let's go a step further. What we want is to  add more information about where the read model data comes from. Since changes are expressed as events, the source of data for read models can only be events.

In this particular case, we already have a relevant event `Organization Created` that we've identified in the [previous tutorial](http://blog.sapiensworks.com/post/2016/08/27/Domain-Map-Tutorial-Create-Organization) . Click on "+" icon from the `Update by` box then select that event. Now we know where to look for that data and you can have as many events as you need. 

![read model 2](http://i.imgur.com/2cjJaGe.png)

This is also, one of the reasons why we model first as much as we can, and only after that we think about implementation. Knowing the read models, their structures and their connected events tells us a great deal of information about how to design and efficient persistence (read) model.    

As a bonus, if you go to `Organization created` details you'll see the list of read models updated by the event. This is very useful if in the future the event structure changes (you'll know at a glance how many read models will be broken :P ).

![event](http://i.imgur.com/3260wq1.png)

You can check out the result of this tutorial [here](http://www.domain-map.rocks/sapiens#/tutorials/assets/f010d912-c8cb-40bd-a407-872b41d422ac/2).






