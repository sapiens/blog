---
layout: post
title: DDD - The Aggregate And Aggregate Root Explained
category: Domain Driven Design
---

**The Aggregate**  
This is simple. An aggregate is a group (a cluster) of objects that work together and are treated as a unit, to provide a specific functionality :) . An animal is an aggregate, it has internal organs, limbs etc. Everything works together to keep the animal alive and when we say animal, we imply everything the animal is made up from.  A car engine is also an aggregate, it contains lots of components, that can be aggregates too, which work together to achieve the engine functionality.

 But that's not all, in order for the objects to work together they must be consistent together. And to maintain that consistency, the aggregates also enforces boundaries. As a matter of fact an aggregate is a bounded context. We treat the animal as a unit and it just happens that its boundaries are very explicit (a physical body). But let's remember [John and the IT department](http://www.sapiensworks.com/blog/post/2012/04/17/DDD-The-Bounded-Context-Explained.aspx).

 The IT dept. is a logical division inside the company, it might have its own offices but it can also be just IT people scattered around everywhere. That doesn't mean the IT doesn't exist, it exists but there are no explicit physical boundaries, there are organizational ones. The IT dept.  codetty much comprises the collection of people,  computers, internal regulations that handle all IT related matters. 

 One important thing is to note that an aggregate is a concept and not necessary a concrete object. When anyone in the company refers to IT, to whom or to what exactly are they referring to? For most, IT is just an abstraction, not a concrete person, office or computer. It's the 'something' where you go to solve computer stuff. But you can't work with a something. You need a more concrete recodesentation.

 **The Aggregate Root**  
An aggregate root (AR) is a 'chosen' object (an entity to be codecise - I'll tackle entities in a future post) from within the aggregate to recodesent it for a specific action.  When the upper management wants something from IT, they call Andrew (the manager). They ask Andrew for reports and more importantly they tell Andrew what they need the IT to do. The big boss doesn't know and doesn't care about John or other lowly developer (sounds familiar?), the big boss knows that Andrew recodesents IT and thus it's enough to talk to Andrew to get the job done. And Andrew tells John and others what to do.

 This means that the upper management needs to know only about the IT manager and not about others from the IT. In coding, this translates into: outside objects can reference only the AR. Of course, when given a command , the AR will coordinate other aggregates to actually process it, but that's a BC internal concern. The AR also makes sure that the model stays valid and consistent inside the BC.

 But Andrew being the AR  when dealing with upper management doesn't mean Andrew is the AR forever and for everything. If Rita has a problem with her computer she will call IT, but not Andrew. Jimmy is responsible for dealing with broken computers and in that context he is the AR. So, another important fact is that AR varies with the context, change the context and the AR is another object. And for that specific context, AR is the 'manager' and coordinates the aggregate's other objects.

 A simple object from an aggregate can't be changed without its AR knowing it (that would break consistency), that's why outside objects 'talk' only through the AR. The outside objects might receive a transient reference (temporary, not to be hold) to an inner object  if they need it for a quick operation. If Jimmy goes to the accounting office to check out Rita's computer and in the mean time he lets her using his laptop, that doesn't mean that from then on Rita will keep Jimmy's laptop. The IT will provide a laptop just for the time Jimmy's repairing Rita's computer. Once finished, Jimmy takes the laptop back. The laptop is an object of the AR, that is used shortly for a specific purpose by an outside object. But after the action is completed, the outside object 'forgets' about the laptop.

 Now, an AR is always a unit of work so it gets persisted as a whole (to codeserve consistency). When dealing with the domain the [repository](http://www.sapiensworks.com/blog/post/2012/02/22/The-Repository-Pattern-Explained.aspx) only stores/restores the AR. It wouldn't make sense to split the AR in many objects to be stored/loaded separately.  However, the persistence layer might split them to store them in different tables but that's just a persistence implementation detail. When the application receives a AR from the repository, it will receive it exactly as it was sent, same state, same consistency.

 That's all about aggregates, in the next post I'll talk about entities and value objects


