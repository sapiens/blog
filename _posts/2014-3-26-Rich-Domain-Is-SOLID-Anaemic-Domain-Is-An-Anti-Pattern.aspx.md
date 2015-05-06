---
layout: post
title: Rich Domain Is SOLID , Anaemic Domain Is An Anti Pattern
category: Architecture
---

Reading that [The Anaemic Domain Model is no Anti-Pattern, it’s a SOLID design](http://blog.inf.ed.ac.uk/sapm/2014/02/04/the-anaemic-domain-model-is-no-anti-pattern-its-a-solid-design/) I couldn't help but to disagree with the author, hence this post.

 I think the author misunderstood how a rich domain (or an anaemic domain) really looks like. It's not about putting everything in one class, not caring about proper layer separation, it's about modeling properly the business concepts and use cases.

 
> "In an RDM, the domain service layer is either extremely thin or non-existent [20], and all domain rules are implemented via domain models. The contention is that domain entities in a RDM are then entirely capable of enforcing their invariants, and therefore the system is sound from an Object-Oriented design perspective."
> 
>  
 Yes, but it's about business rules that the concept needs in order to be valid. It's not about the business rules required by a use case. Those are treated at the use case level, that is the Application (Services) Layer which isn't that thin anymore. As an example of a rule that should be enforced by a domain concept, an order should not accept quantities or prices lower than 0 (well, to be more exact the Price object used by the Order shouldn't allow values < 0).

 Next, I simply can't agree with the rich domain example codesented by the author. ActiveRecord pattern shouldn't be used in a rich domain context. Ever! You want to keep things decoupled so, no way a proper business object should contain db access code. The example is simply flawed. A DomainEntity base class containing CRUD methods is something belonging to persistence (ORM entity), a real domain base class doesn't care about database, even less about CRUD.

 
> "A proponent of the RDM architecture might claim that the hypothetical example provided is not recodesentative of an true RDM. It might be suggested that a well implemented Rich Domain Model would not mix persistence concerns with the domain entity, instead using Data Transfer Objects (DTO’s) [18, 17] to interface with the persistence layer."
> 
>  
 Actually, it will use a repository. The DTOs might work for querying purposes but that's it.

 The anaemic domain example codesented looks codetty SOLID if not a bit over engineered. However, that's a use case that's using the domain concepts. I get the feeling people think that rich domain object have to be big, complex objects with 20 public methods and 17 children and even more private methods. The richness of a domain stems from the number and complexity of concepts(with relevant behaviour) and use cases (business rules, business flows), it's not about fat objects.

 
> "As the hypothetical RDM is refactored to apply the SOLID principles, more granular domain entities could be broken out; the Customer domain entity might be split into CustomerPurchase and CustomerRefund domain models. However, these new domain models may still depend on atomic domain rules which may change independently without otherwise affecting the domain entity, and might be depended on by multiple domain entities; to avoid duplication and coupling, these domain rules could then be further factored out into their own modules and accessed via an abstract interface. The result is that as the hypothetical RDM is refactored to apply the SOLID principles, the architecture tends towards the ADM! "
> 
>  
 No it doesn't! CustomerPurchase and CustomerRefund are use cases of the Customer concept (entity). If another bounded context sees the Customer a bit differently(structure or behaviour), then we're talking about a different concept (the model is valid only within the context boundaries, it shouldn't be reused under the codetext of DRY).

 Concepts interact with other concepts (behaviour) according to the use case. The behaviour is a part of the concept and cares only about the concept's limited subdomain. And that behaviour certainly isn't related to persistence or UI widgets or even other bounded context (outside the concept's context). You can say an use case encapsulates how a concept 'works' with other concepts given a certain context. But that **behaviour is intrinsic to the domain object**, the use case just uses it.

 In a properly modelled _rich domain_, a service (implementing a use case) manages 1 or more domain concepts towards a goal. But the service doesn't define a (new) behaviour for the involved objects (the concepts don't change). In an _anaemic domain_, the 'domain' model is just data while the behaviour is defined in another place (a service), not only keeping the data separated from the behaviour but also mixing together (confusing) behaviour and use case, which leads to bad modelling hence it's an anti pattern. And improper modelling means poor maintainability, complicated codebase etc.

 When dealing with any domain (rich, simple) you should be in complete control over the code. You should know WHY you defined a class in a certain way, in what layer are you working, what is the responsibility of a certain class and if the class really models the proper business concept. It requires a lot of attention and analytic skill but it's not hard. The biggest mistakes you can make is to be superficial and to get bogged down by technical details. Remember that domain modelling is done at a high level, so the concern of lazy loading or the debate to use a List or an IQueryable don't belong here.


