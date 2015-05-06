---
layout: post
title:  Domain Driven Design Modelling Example - Brain and Neurons
category: Domain Driven Design
---

I got this question on stackoverflow

 
> A brain is made up of billions of neurons, so if you have brain as your AR you potentially have a performance issue as its not feasible to retrieve all neurons when you want to activate some behaviour of a brain. If I make the neuron the AR, then its not trivial to validate its behaviour and there is the issue of creating new neurons within a brain. Which should be the AR, or should there be some other AR to encapsulate these
> 
>  
 I found the use case quite interesting because of the fact that we're dealing with billions of "children". So is the Brain an Aggregate Root (AR)? Is the Neuron a Value Object (VO) or an AR? Assuming that we want to model things as close as we know how the brain works I'll say it quick: Brain is an AR, Neuron is a VO and the Brain will always require ALL the neurons (yes, billions of them) every time you ask for a Brain object from the repository. So I'm saying to load billions of rows anytime you need a brain (sic). Maybe I need a brain too. Or I'm just a good Domain modeller. Here's how it is:

 A brain can't function without neurons, a brain **needs** the neurons. But does it need all of them? Well, let's see... I'm telling the Brain to learn a poem. The Brain will use some neurons to store that information. I tell the Brain to solve a math problem. The Brain will use the same or other neurons to do that. Now, do I (the client using the brain object) know which neurons are used for what behaviour? Or is it an implementation detail of the Brain? That's the point, only the Brain knows how to use the Neurons. It groups them in regions and it might use 1 or 10 regions for one single task. But nobody outside the Brain knows these details. This is proper encapsulation.

 So when you need a Brain you get it whole with all neurons no matter how many they are. You don't get only parts of it, because they don't make sense independently. Btw, the neurons probably are distinct for the Brain but that doesn't mean they are automatically Domain Entity. After all, maybe the brain doesn't care which neurons are used. Maybe it cares more about regions and it just tells a region to supply a number of available neurons. Implementation details...

 Should the Neuron be an Entity or even an AR? Maybe, but not in this Aggregate. Here, it's just something the Brain uses internally. If you make it an entity or an AR, you'll have something with an information that only the Brain can understand. And if you understand the neuron information then you don't really need the Brain. In this aggregate, Neuron is at most a VO.

 But what about the technicalities? I mean there are still billions of neurons. It's a challenge to load them all, right? Isn't lazy loading the perfect solution here? It isn't, because the ORM doesn't know which neurons the Brain needs. And the brain needs to have some virtual properties (let's say the ORM works with private properties) which should load only specific neurons. But what do you do? You create one property for each possible behaviour so that you can load _only_ those specific neurons? And if the Brain doesn't care about a specific Neuron but only about the signature of the data contained, how can you implement that for the ORM to work with? Lazy loading has just complicated things. And this is why lazy loading is an anti-pattern for Domain entities, it has to know domain details in order to know what data to retrieve. And not everything can be put in a virtual property (to allow the ORM to create the proxy), not if you want to make the object maintainable.

 The solution here is to load everything the Brain needs to do its job, because it's how the brain works according to the Domain (it needs ALL the neurons) and it's a maintainable solution. Maybe not the fastest one, but you're dealing with a complex notion where maintainability matters more and it isn't like you're using the same object for queries, right? And it's not about the number of neurons, it's about brain's data size. Do be aware that in this case, the Brain and Neurons have a direct relationship in a way that one can't work or it doesn't make sense without the other.

 Other cases might resemble this one, but the relationship can be one of association not definition. In those cases, you have grouping criteria and items ([forum and threads](http://www.sapiensworks.com/blog/post/2013/01/15/Domain-Driven-Design-Aggregate-Root-Modelling-Fallacy.aspx)) or different concepts working together ([table and chairs](http://www.sapiensworks.com/blog/post/2013/10/18/Modelling-Aggregate-Roots-Relationships.aspx))


