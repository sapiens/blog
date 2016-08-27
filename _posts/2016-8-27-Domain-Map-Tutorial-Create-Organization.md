---
layout: dmap
title: DDD Tutorial - Modelling "Create Organization"   
category: Domain Map Tutorials
---

# Business functionality: Create Organization

We want to create an organization, similar to what GitHub has. An organization is a group of members who can participate in the organization's projects. Organization name must be unique, at most 50 characters length. An organization should have at least 1 member in admin role. A user can be part of any number of organizations.
 

# Modelling

## The bounded context

First things first, we need to identify the bounded context (BC). We're dealing with users and groups so, I'd say the proper bounded context should be something like `Membership`. This is not core domain functionality, but I prefer to look at the whole app as a group of bounded contexts (anything that has a model with boundaries is a BC). Some of them belong to the Domain, others are just application specific. The `Membership` is app specific, but that doesn't change the fact that it has its own specific model that needs to be maintainable.

At this point we need to create the `Membership` BC. 

![create context](http://i.imgur.com/wmJTKiS.png)

Click on the add icon then write the name 'Membership' then press the Enter key. We've created and switched to the new context.

## Using CQS

We need to identify the type of the business case: command or query. In this case, we're dealing with a command, since the result of the action changes the existing business state. Let's create the command case. Click on the blue `Add` button (left of tabs). 

![create command](http://i.imgur.com/RjnRK90.png)
![write name](http://i.imgur.com/oir7tGF.png)

Press 'Enter' or click 'Create'. The command case is created.

## Identifying the event

Any business state change is represented by an event. In this case the change is that an organization has been created, so we create our `OrganizationCreated` event.

![create event](http://i.imgur.com/sGlUfrv.png)

Next, click on the `Events` tab, then on the `Organization created` event. Now, it's time to find out the change details i.e the event structure. In this case they're simple.

![event details](http://i.imgur.com/67tJtTa.png)

Click 'Save' when finished.

## Identifying the concepts and the aggregate

We know the command, we know the result, now it's time to find out the what are the models involved in generating that change. It's pretty obvious that we have the concept of `Organization` which is an [Entity](http://blog.sapiensworks.com/post/2016/07/29/DDD-Entities-Value-Objects-Explained). Let's create it.

![event details](http://i.imgur.com/V1jmEMk.png)

Now, we need to identify a model _relevant_ for our business case. Click the `Command Use Cases` tab, then click on `Create Organization`. Press the '+' icon in the `Uses Command Model of` box. It should look like below

![link command to entity](http://i.imgur.com/n9sv5nG.png)

Click on `Organization` to link the command case to the concept in charge of doing changes. Next, click on the `Organization` link to go to the aggregate which is specific to that command.

![model link](http://i.imgur.com/YyEHAhr.png)

Now, it's time to determine the aggregate composition (relevant to this business case, only!) and business constraints. The important thing here is to understand that we're identifying a model, we're not designing a class with properties. In our case, we know that the organization needs a name and an admin. We start with the name, by creating a Value Object (by clicking Add-> Value Object). Once we have it, we link it with the aggregate ( click on the '+' icon in the `Aggregate composition` box).

![aggregate to value object](http://i.imgur.com/yDLSv1d.png)

Now, let's write the business rules for the organization name. Click on it to go to the Value Object's details. 

![value object details](http://i.imgur.com/tei5NzR.png)

I wrote the constraints as a regex. Feel free to write things how you want them as they communicate the rules that the values must respect. Click 'Save' then click on `Create organization` link at the right to navigate back to the aggregate.

Now, we know that the organization _must_ have an (authenticated) user in the admin role. This is a [strong relationship](http://blog.sapiensworks.com/post/2016/08/24/DDD-Relationships) between the `Organization` concept and the `Member` concept. We're going to represent this relationship by making the aggregate reference the `Member` entity, which doesn't exist so we have to create it first (Add -> Entity). Then, click on "+" in the `Aggregate composition` box to link the `Member` concept.

![aggregate details](http://i.imgur.com/Mx0nOQc.png)

You see that we have that "All are required" rule, this means all the components of the aggregate must be present in valid form. When writing code, the `Member` reference will be implemented as a `MemberId` value object. 

There's no mention of the "organization name must be unique" rule. That's because it's not an aggregate rule, but it's still a rule that should be enforced. At this moment, Domain Map doesn't have any specific tools for business rules that aren't part of an aggregate or a [Domain Service](http://blog.sapiensworks.com/post/2016/08/16/DDD-Domain-Services-Explained) but we still want to record this information.

So, we'll create a _Note_ , named "Other rules".

![create note](http://i.imgur.com/tS9vjQJ.png)

You can create notes to record anything you want, including implementation details. 

![note details](http://i.imgur.com/0Z0sWSR.png)

Close the note and scroll down a bit. You should see the `Outcomes` box, with a `Default Outcome` option. Click on it, then click on the "+" icon from the `Events` box. Click on the `Organization created` event to specify the default outcome (change) generated by the aggregate.

![outcome](http://i.imgur.com/r4OMyY8.png)

Since an aggregate **controls** change and the change is represented as an event, we always want to know which aggregate generated which change. In this case we only have one outcome, but in other cases, we can have multiple outcomes, each with their own event(s). This association also allows us to navigate to event details and back. Feel free to navigate around a bit, clicking on entities, value objects etc.

# Conclusion

And that's it! We've identified the relevant concepts, models, events and rules for our command business case ([click here](http://www.domain-map.rocks/sapiens#/tutorials/assets/8832ca49-95bb-44c3-ae24-522f24dbf1ed/1) to see the result in Domain Map). This is information **only**, it has nothing to do with implementation. But once we have all this, it's easier to write the code however you like it. The point is, the model stays the same regardless of programming language, framework or programming style. With Domain Map, the focus is on gathering and recording domain knowledge, not writing code.

In the next tutorial, we're going to model the "List organization members" business case. 


