---
layout: post
title: DDD Decoded - The Aggregate and Aggregate Root Explained (Part 1)
category: Domain driven design
redirect_from: "/post/2012/04/18/DDD-Aggregates-And-Aggregates-Root-Explained.aspx"
---

For easy reading this topic is split in 3 parts: theory, [example modelling](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-2) and [coding (C#)](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-3) .

A lot of actual and virtual ink has been used to explain this important DDD concept, but as Vaughn Vernon puts it "aggregates are one of the most important DDD patterns and one of the most misunderstood ones". That's because they are hard to explain, but once you've _really_ understood it, everything becomes easy and clear. But, first you have to accept the fact that [DDD is not about coding](http://blog.sapiensworks.com/post/2015/11/23/DDD-is-not-programming). Once you know that DDD is just a way to gather domain information and organize it in a technical (developer) friendly manner, you're ready to grok the Aggregate and its sibling _concept_, the Aggregate Root.   

# A model specific to one business case

Everytime we change something we have to make sure we're making valid changes from the business point of view. Now, the easiest way is to represent things as close as we can to how the business sees them. This means we have to identify the **concepts** that the domain uses, but more importantly, we need to identify a _relevant_ representation of a concept, relevant for the busines case using that concept. 

For example, I can identify the `Invoice` concept, but what I really need is a representation of it (a model) that I can use in a specific use case like: `Create invoice`. But what about `Cancel invoice`? Well, it has its own represenation of `Invoice`. Each business case needs its own _relevant_ model even if it involves the same concept. We end up with one model per use case but with multiple representations of the same concept. And yes, we **want** that. _Relevancy_ is a keyword in DDD.

## A CQRS intermezzo

Applying the Command Query Segregation (CQS) principle, we ask ourselves: "Am I trying to **change** things here? Do I need this model to change the existing business state?". If yes, then our model is actually a Command Model (yes, that's the C from CQRS) that will contain all he information we need to change that particular part of the business state. 

For a `Create Invoice` case, we need a model of `Invoice` that makes sense to that business case _only_! The domain expert will tell you what is the data and the business constraints needed to create an invoice, basically, the **components** and the **rules** which together define a new invoice. We've identified our Command model which in DDD is called an Aggregate!!

But before we continue...

## A real world aggregate example 

Let's say you want to buy a table from Ikea. Do you get a table? Not really, you get a bunch of components and instructions in a box that you have to put together yourself. Let's say you take out all the wooden parts, screws etc from the box and put them in a pile. Do you have a table? No. You have the **components** of a table but you need to _assemble_ them according to some **rules** in order to end up with a table.

The table is an **aggregate**, that is a group of components 'held' together by (assembly)rules, in order to act as a **single** thing. Both components and rules (Vernon calls them invariants) are what makes an aggregate. A pile of parts doesn't make a table, just some assembly rules don't mean anything, we need all of them to create our table. 

The aggregate is a model that represents all the relevant information we need to change something. Only that, the information is organized into components, themselves models of other smaller concepts, and rules that needs to be respected. Everytime we want to make changes, we need to identify the aggregate that tell us the relevant components and the rules they must respect. 

# Identifying an Aggregate

Back to our table example. First thing, we need a busines case that aims to make changes. That's important, because if you only need to read stuff, you don't need an aggregate, but just a simple read model and remember that an aggregate is **always** a command model (meant to change business state). While we don't really want to build a table, that's what we need to do in order to have one, so this is our business case: `Assemble table` (note the semantics).

We want to identify the command business cases as granular as they can be and this means we want one business state change per case. Many times, we think it's one case when in fact it's a whole process i.e a sequence of cases. So, we need to pay attention to the domain expert, they can help us identify the boundaries of a business case. **Example**: a process named "Generate invoice" can involve the cases of "Create invoice" and "Create PDF from invoice". If we need to change the state of different concepts ("Invoice","Pdf Invoice"), then we might deal with a process. And a process can span multiple bounded contexts as well. But we want to identify only one business case (or at least we tackle them one at the time) i.e one relevant business change.  

Then we identify the business concept - or domain relationship - that needs to change (being created, updated, deleted) and the relevant model representing it. Again, we want a model which is specific for that business case. So we end up with a bunch of other concepts (we need their models, too!) and business rules. So, for our table, which is the identified concept, we need a representation that tells us what are the important parts and rules required to build it. In this case, we're lucky, Ikea did the work for us, so our model consists of all the wooden parts and screws, plus the assembly instructions.

## An aggregate is an always consistent portion of the business model  

An aggregate defines consistency boundaries, that is, everything inside it needs to be immediate consistent. This is important, because it tells us that no matter how many actual changes (state mutations) need to be performed, we have to see them as one commit, one unit of work, basically one 'big' change made up from smaller related changes which need to succeed together.

But we don't actually _design_ things to be in a consistent manner; the fact that we have all those components and rules **together** tells us that we're dealing with a group acting as a single unit that needs to always be consistent. This is how we know we've _found_ an aggregate.

We can say that we can identify an aggregate either starting from a 'big' concept and understand its (busines) composition, or by noticing related concepts and rules that together define an area that needs to be always consistent. Either way, we end up with an aggregate.  

A somewhat interesting situation is when we deal with domain relationships, in some cases we need to identify an aggregate for them too. But that's a topic for another post.

As you see, modelling an aggregate has nothing to with object oriented design, but everything to do with paying attention to the domain expert and really groking the domain. 


## The aggregate role is very specific

This is an important thing. We need a model because we want to make valid business state changes. However, the purpose of our aggregate is to **control** change, _not be_ the change. Yes, we have data there organized as Value Objects or Entity references (stay tuned for a post about these) but that's because it's the easiest and most maintainable way to enforce the business rules. We're not interested in the state itself, we're interested in ensuring that the intended changes respect the rules and for that we're 'borrowing' the domain mindset i.e we look at things as if WE were part of the business.

An aggregate instance communicates that everything is ok for a specific business state change to happen. And, yes, we need to persist the busines state changes. But that doesn't mean the aggregate itself needs to be persisted (a possible implementation detail). Remember that the aggregate is just a construct to organize business rules, it's not a meant to be a representation of state.

So, if the aggregate is not the change itself, what is it? The change is expressed as one or more relevant **Domain Events** that are generated by the aggregate. And those need to be recorded (persisted) and applied (interpreted). When we apply an event we "process" the business implications of it. This means some value has changed or a business scenario can be triggered. More about that in a future post.   

You can say the job of the aggregate goes like this: "Based on the input you gave me and business rules that I know, the following business state changes took place: X happened with these details. Do whatever you want with it, it's not my job, I'm done here".

# The role of Aggregate Root

The companion of the Aggregate is the Aggregate Root (AR) and we already know that we should use the AR to manipulate things in our aggregate. However...  

## Artifacts everywhere..

In the beginning, DDD was very much mixed (coupled) with OOP. And that's why we usually have an OOP centric view and patterns names. But today, DDD is about identifying a domain model regardless how it will be implemented; however its OOP roots(ha!) left some artifacts, especially in naming e.g Value Object.

The name "Aggregate Root" make sense in an OOP approach where you have a group of objects and you want only one to be the "root", the facade representing the whole structure, however, in a more abstract manner, the role of the AR is simply to enforce the aggregate's business/consistency rules. So today, AR is a **role** that can be implemented by an object or just a function. If the aggregate means a group of components and rules, the AR is the "guardian" making sure we're dealing with the right components and the rules are respected. In our real life table example, the AR is the person assembling the table.  


And that's the theory! Onward to a [modelling example](http://blog.sapiensworks.com/post/2016/07/14/DDD-Aggregate-Decoded-2).
 



