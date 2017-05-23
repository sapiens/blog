---
layout: post
title: DDD Tutorial - Modelling Domain Relationship and Domain Service
category: Domain Map Tutorials
---

This tutorial is an excerpt of the _actual_ DDD modelling process of [Value++](http://www.getvpp.com/) (Agile project management software) using [Domain Map](http://www.domain-map.rocks/).         

# Business functionality: Remove a user from an organization

A quite straight forward functionality, however there is a little catch: an organization should _always_ have at least 1 admin left. Basically, if the user is the last admin, the operation should fail (with an error message). 
   
# Modelling

## The bounded context

We're dealing with users and organization so, the relevant context is `Membership` (switch to it).

## Applying CQS

The functionality clearly changes the business state, so we are in a **Command** case. Let's create it in Domain Map, Add -> Command Use Case -> enter "Remove user from organization".

## Identifying the business state change

Well, the resulting change is about a (site) member who got removed from an organization. Add-> Event -> enter `MemberRemovedFromOrganization` . That's our event and we need to find out the relevant details. Obviously, we need to know the member id and organization id. However, we need one of them to be the [entity](http://blog.sapiensworks.com/post/2016/07/29/DDD-Entities-Value-Objects-Explained) id, basically the id of the concept that the change is associated with. But both `Member` and `Organization` concepts are entities, so _who is_ in 'charge'? What concept 'cares' more about the operation?

It turns out the operation makes most sense for the `Organization` concept, so, the entity id will be the organization id. Click on the `Events` tab, then click on the new event to enter the details we've found out. 

![event](http://i.imgur.com/07uCJ68.png)

## Identifying the aggregate 

We know the result, but we need to enforce the business rules. First, we need to identify the aggregate, but in this situation we're dealing with a [weak relationship](http://blog.sapiensworks.com/post/2016/08/24/DDD-Relationships), we are 'destroying' an instance of that relationship and the aggregate should reflect that relationship, not `Member` or `Organization`. At this point, we don't have that relationship represented in Domain Map so let's create it now.

Click on the `Entities` tab, and then click on `Organization`. Then click on the "+" icon from the `Relationships` box.

![dialog](http://i.imgur.com/8RE7Tvl.png)

Click on "+" from `With Concept` box then select `Member`. Let's name this relationship "Members can be part of an organization". 

![dialog2](http://i.imgur.com/Z2lDw9S.png)

Click "Create" and the new relationship should appear 

![dialog3](http://i.imgur.com/tyPcruc.png)


This relationship will appear on both `Member` and `Organization` details and the whole point is to acknowledge that these 2 concepts can be (dis)associated. However, this is **not** an aggregate, we still have to identify it and we do that as part of the current business case.    

Click on `Command Use Cases` tab then click on `Remove user from organization`. Click "+" from `Uses command model of` and select the newly created relationship. What this tell us is that the command case will **change** an instance of the relationship and in this case, we 'destroy' it. And this aggregate will be in charge of that. Click on it to go to the details.

Inside the aggregate, you'll see both (implicit) concepts: `Member` and `Organization`, then in the default outcome, select `MemberRemovedFromOrganization`. 

![agg](http://i.imgur.com/VbBTiyJ.png) 

We're done with the aggregate.

## Using a Domain Service

But that's not all! Remember the business rule that the last admin of the group must not be removed. So we need to enforce that and the best place is a Domain Service (DS). Why not a part of the aggregate, you ask? Because this rule is unrelated to the consistency of the relationship. It is an 'outside" (the aggregate) rule so, we make it part of a DS. Even more, it's a 'permission' kinda of rule, basically it tells the application service if the business case can continue. If not, it doesn't even get to the aggregate part.

Add -> Domain Service -> enter: `Can user be removed from organization` . Then, go to the command case, click on "+" from `Involves` box to link to the domain service. Then click on the service name link to go to its details. In the `Business behaviour` box, write all the details that we need.

![service](http://i.imgur.com/UCTvcVz.png) 

Well, I've kinda included a bit of implementation details here but hey, it doesn't hurt anyone, especially if only developers (a domain modeller is usually one) will see it. But if you want to go even deeper with coding details (like "we should use a couple of query objects etc"), I suggest to put that in a note (the `note` icon next to the name).

# Conclusion

This was a more complex case, but in the end we have our command case which involves a domain service (with all required details), we're aware of a relationship between concepts and we know the aggregate and the event. Best part? No specific code was written, but with all this information available, it's easier now to write an implementation using your favourite stack.

You can check out the model from this tutorial [here](http://www.domain-map.rocks/sapiens#/tutorials/view/8832ca49-95bb-44c3-ae24-522f24dbf1ed/ac/93f8b2b3-f106-4980-bda5-10749febe745).






