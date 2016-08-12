---
layout: post
title: DDD Decoded - Bounded Contexts Explained
category: DDD Decoded
---

In DDD, we organize the Domain into Bounded Contexts(BC), with at least 1 BC. An important pattern, the BC is a bit tricky to "get" and to explain, but I assure you that you _need_ it for proper DDD and it simplifies things on the long run.

# What is a Bounded Context?

What if I told you that you've been using the bounded context principle for a while, without knowing it? You know those UI, Business and Database/Persistence layers? Those are literally bounded contexts, an implementation of the **separation of concerns** principle. A complex Domain can be considered a bunch of functionalities, but we like to group them into 'specialized' modules, where each module is responsible for a high level concern.

If the Domain would be a company, the BCs would be its departments: accounting, HR, marketing, production etc. Each with its own responsibility, different but working together. And since DDD is about being **Domain** driven, we want to design our application similarly to how the real domain(business) is organized. 

# Why do we need a Bounded Context?

It's more than just mimicking the business' structure. The keyword here is **bounded**, that is, each BC has boundaries, limits that isolate the model from other BCs(if there are more than one). Actually, the boundaries _are_ the main benefit of a BC because they 'contain' a specific model, allowing us to focus on it. We can say that the main utility of BC is that it allows **better decoupling**, which is the most important trait of maintainability. Basically, we group together related models while keeping them separated from other models. 

And like everything else in DDD, we aren't the ones designing or deciding BCs, we _identify_ them. The business already knows what responsibilities and limits each department has, we're just putting that information together. 

# How we identify a BC?

Well, we pay attention to the language. The ubiquitous language is not just a DDD buzzword. It means that the business language has the same meaning in a certain context. `Invoice` means something to Accounting, but it can be just the name of a class for a programmer. Just because the name is identical doesn't mean it's the same concept. And Marketing can say `Customer` and Accounting can say `Payer` but they can both mean the same thing for the business as a whole. 

The important part here is that names are by themselves irrelevant. We need to understand the _concept_'s definition that makes sense in that context in order to place a business case in a certain BC. Many times is quite easy, `Create an order` is part of `ECommerce` but sending the order's confirmation email is clearly part of a different BC, something like `Marketing` or `Sales`. _How_ you send the email it's an implementation/infrastructure detail and we're not interested in that (at this point).

# How do the BCs communicate?

Since the model of a BC makes sense only inside that BC we can't just use (refer) other BC's model directly. But in many cases we need information available in another BC. How can we get it, without breaking the boundaries? 

Enter **Domain Events**. They represent changes of the global business state, but they do have another, very useful technical trait: they are messages, simple data structures. They are no longer a _model_ , they are simply information and they don't belong to a BC, they are domain wide valid.

If one BC is interested in data from another BC, it will just subscribe to the relevant events and then it will use that data to maintain its very own **read model** (yes, that's CQRS in action). And since messages are data and not part of a model, boundaries are not violated. All this should be part of the BC's own infrastructure.

But what about invoking a business case which is part of a different BC? When you need that, it means you're dealing with a business process, a sequence of business cases, however, even if a business case belongs to a BC, the process itself doesn't and we're going to have a **process manager** a.k.a saga in charge, which is a technical implementation detail. When we identify a process, we care about which business cases are part of it, the manager itself doesn't exist as part of the domain. 

A business case shouldn't invoke directly the next business case i.e it shouldn't act as a part-time project manager; in order to achieve decoupling and to maintain boundaries, business cases don't know anything about other cases, it's the process manager's job to know and to invoke the next case.

## In special situations...

..you might want to couple one BC to another, because it's the easiest way. It can be a valid, good enough solution, but you have to be careful. The trick here is to find the right compromise between decoupling and easy implementation.

# Conclusion

The **Bounded Context** concept is very important for the maintainability of the app. It allows us to deal with relevant models that don't overlap. It's important to note that this is a logical grouping criteria and we can implement everything inside a monolith. But if the app needs to be scalable and we go distributed, then we may want to implement one BC per app/process/server. 



