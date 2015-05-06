---
layout: post
title: DDD - Entities And Value Objects Explained
category: Domain Driven Design
---

There's no DDD if there are no entities or value objects lying around, eh? Before we continue, let me clear up a thing: Entities and Value Objects (VO) are concepts. Depending on the context, an object can be both and its implementation can be slightly different.

 The point is you don't start with: let's find the Entities and each entity will be a subclass of an abstract Entity class. You try to model a problem or a solution using objects. The fact that in a context one will be used as an Entity will simply emerge when you design the model. Very little matters for the app that an object is an Entity, however it might matter when you communicate with other developers.

 
### The Entity

 An object with an id is an entity. That was simple, wasn't it? You put an id in a calss, when you still want to refer to a certain object even if all the other variants have changed. That's why yoy put an id, the id never changes and you know that you will always get the same object when you ask for the User with Id = 8. The user object can change the name or email properties, in fact it can change all the properties it will still be the same User with id 8.

 
### The Value Object

 This is an object without id. If any of its variants changes, the object is considered to be another object. Most of the time a Tag is a VO. Change the tag name and suddenly you have another tag. That's because the object's identity is its value (hence the name Value Object).  One thing to note is that is good practice to make the VO immutable, that is once it's created it can never be changed. That doesn't mean it can't have behavior. But that behavior can't change state, it will just return another object.

 You may wonder why you should use VOs. The Value Objects are ideal to encapsulate bits of domain. If something is a VO, I know that something is valid and immutable. An Entity might change state at any time but the VO is always 'stable'.

 
### Is it my object an Entity or a Value Object?

 That's the most tricky part and here is where everyone has problems. But here's a tip that will save you lots of headache: a concept from the domain can be an entity or a VO depending on  the context. This means that before you decide that object X is an entity, first you have to identify the BC where object X will be used . You'd be surprised how often you'd be using Value Objects.

 You have a car, which among other things has 4 wheels and a engine. Is it the engine an entity ? Well, let's suppose the car works with the engine type ENG-3. When the engine breaks, can you replace it with another engine of type ENG-3? Pretty much yes. In this case an engine is a VO. That's because you don;t care about that specific engine, you care only about an engine (any engine) that is compatible with your car.

 But, if you're a car engine manufacturer, you want to keep track of each engine produced. In that case, the engine is an entity as the company cares about each specific engine. If you built your own engine for your car, then the engine is an entity because even if it's broken, you won't replace it. You'll repair it and change its components but it will still be the same engine.

 You buy a book form the library. If you lose that book and buy another one and you don't care which is which, then the book is a VO. But if the first book you buy is signed by the author (a very limited edition), then it's very hard to replace it. You care about that specific book, another simple copy won't replace it. That book is an entity.

 You have a heart. You do care about your heart but you can replace it with a compatible heart and your body can function further. The heart is a Value Object. You, as a person, are you an entity? Well that depends too. For your friends and family you are, no matter what you change about yourself is still you.

 However, for your employer you might be just a VO. If your employer just need someone with C# skills, anyone who knows C# can qualify. Today is you, tomorrow will be another person (perhaps with a lower wage). In this case, the developer is a VO. But if you're John the star developer, you are an entity. John can't be replaced with another developer. John matters for the company because he is John. The big boss knows about John, not about a C# developer.

 As you can see, everything matters according to context, that's why it doesn't make sense to start modelling by saying: I have this  entity. You have an object and the domain context will reveal if it will be treated like an Entity or like VO. A very BIG confusion happens because of mixing layers. Especially when using an ORM, there you can have Entities which are a detail of the ORM. In other cases you might want to store some objects using an id. But that id makes sense only when dealing with the db (a different context). The Tag is a VO for the Domain, but it's an entity for the Persistence.

 So design your objects to form a valid model only for a specific context. Just inheriting from an abstract Entity class doesn't mean that object **IS** a real Domain entity and if that base class doesn't have some common behavior useful for ALL entities, **don't do it**.


