---
layout: post
title: Using DDD, CQRS, Event Sourcing With A Vague, Fast Changing Domain
category: Domain driven design
---

If you're a startup, there's a high chance that your product/service is new and disruptive. This means you (or the stakeholders) have just a vague idea of what you want and what can be called Domain is just a mishmash of things that change every day. Is it a good idea to use DDD, CQRS or Event Sourcing in this case?

Yes, it is! It will simplify your work greatly, _if_ you really understand DDD/CQRS/ES . At this point, you might be puzzled how can DDD be a good choice when the domain is fuzzy and there's no domain expert. Not a big problem, because it's all about understanding what the stakeholder want to accomplish, step by step.

Now truth be told, especially in cases like this, you'd be happy if you can get even a half-assed idea, because nobody really knows what they want, it's all about vague, raw concepts and maybe, very crude business processes. But regardless, you can latch on the most important thing: language.

Sure, the domain language is volatile, things will change daily and many times drastically, but the good news is you need to live in the present, not the future. Present concepts, present rules are what matter, we don't care much about 'what if'. So yeah, basically an iron YAGNI.

In order to keep things sane and productive, first step is to see the whole application as a group of (high level)functionalities. For example, for Twitter would be stuff like: register, login, follow, tweet, retweet etc, basically all the app features that can be used by the users(I'm including API here too). So, start identifying **application functionality**.

Next step is, when modelling, to treat each functionality as independent. What I mean here is that the resulting model for one functionality should be decoupled from other models. Applying CQRS, it means that **command models** are _not reusable_, they are _unique and specific_ to each functionality. From DDD point of view, it means we don't reuse Aggregates. This way, each functionality is decoupled and they can evolve differently, according to needs.

What about Event Sourcing? If domain changes a lot, then the events will change too, right? Is it worth it? Of course it is! First of all, **domain events** are notifications of changes that already happened. Their structure may change if details change (but the data itself remains), but when it comes to business rules the changes affect the future, not the past.

Let me give you an example. Chances are you have a degree/certification in something. In order to get that you had to pass an exam that verified that you have certain knowledge. Let's say 10 years have passed. The syllabus changed and the exam now requires any exam taker to know the new stuff. So, anyone passing the exam now will be according to the new rules and content. But what about your 10 years old degree? It stays the same, you don't need to change it, the new rules don't apply to you (the past).

The events are the degrees obtained after the exam (according to at-the-moment business rules). Once you have them,they don't go back to being re-validated when business rules(exam) change. They are old data and yes, their details can change and you add/remove those new fields to make it technical compatible to current version. But in the end, they are just data. When things change, our model change but the existing data stays the same. We should also acknowledge that the event class structure as an implementation of a domain event is not a synonym (but a destination) for the event data stored in the db.

However, this require a certain approach i.e your aggregates should basically be **events factories**. A business object should contain only rules and a transient state that wouldn't be persisted. Simply put, your business objects don't contain business data with rules, only the business rules required to create events.

With ES and CQRS, a specific domain state can be restored as a _readonly model_ by replaying events (which basically means assigning data).

By doing that, you keep things maintainable. The domain model implementation is mainly rules (organized as specific one-use aggregates, value objects and services), while persistence gets only the valid relevant business data. The important thing here is that persistence contains the _whole domain state_ expressed as events streams and projections.

## Simple Example

Let's assume you're building the next Facebook and right now the registration functionality requires only username and password. You'll end up with an `UserRegistered` event.

```csharp
public class UserRegistered
{
    public string Username {get;set;}
    public byte[] PasswordHash {get;set;}
}
```
Two weeks pass and you decide that all new users needs to provide an email address too. This means that you need to change the aggregate to enforce the new rule. Also, the event (class) itself needs to be changed (add property Email). Any new user will be required to provide an email, everything is good. What about the previous users?

This is not a case where we can 'upgrade' somehow existing user info, so basically the app UI will present a popup after login, telling them to provide a valid email. This means adding another event `EmailUpdated` or similar and now, the domain has the emails for the 'old' users as well. That's why we look at the business state (persisted relevant business data expressed as events) as a whole. It can be updated by means of different aggregates and events. The projected queryable model is also updated.

Regarding technical implementation, while the event structure has changed, the existing data is still the same and this means you need to map the old data to new structure (which is the job of the Event Store). As I've said before, old events aren't re-validated by new rules so the question now is, what happens when we replay events and the projected read model.

I'll start with the latter. Using a RDBMS means you need to update one or more tables definition in this case you add a new column `Email` which is nullable. The read model needs to reflect the fact that not all existing users have an email. This is a simple scenario and here things are as CRUD as they can get.

If a db schema changes very much or you're using a nosql db, maybe the easiest method is to regenerate the whole read model.

About restoring an object state by replaying events that contain old data, first of all is important to remember that the restored state is immutable and the events are just sources of data. No business rules come into play, an `Apply` function just 'maps' information from an event to object state. Again, this function should be aware that Email can be empty.

Basically, when the domain changes, we update the model in our app, but because we have and want to keep the old data, the model still needs to account for artifacts of old rules as they affect the current domain state. In our example, email wasn't needed and that generated specific data, then email was added and the model now correctly requires email for new users while it also deals with the implications of the old rule i.e users without an email.

As a domain evolves, the rules keep changing, but the existing business data doesn't. Therefore everytime we deal with a business rule change, we have to consider its residual implications from a business point of view first(i.e non-technical) and technical second.

In our example, regardless of how many times the rules change, it may always be 1 user who registered at the beginning and never returned. So, you either have the concept of an abandoned account that needs to be deleted or, as long as the app exists you need to remember that some users didn't provide an email (business pov) and the Email property/column can be null/empty (implementation).

## Conclusion

Every business changes in time, the only difference is the pace. When dealing with a new, vague, volatile domain the pace is very fast but in the end you'd have the same issues you'd have when the domain changes at a slower pace. It's harder because you have to grow yourself into a Domain Expert but from an application design and implementation point of view, things will be very similar. In the end, what makes the app maintainable is how we've applied the principles behind DDD/ES/CQRS. Recipes, libraries and frameworks are just tools that should be used according to the above-mentioned principles.
