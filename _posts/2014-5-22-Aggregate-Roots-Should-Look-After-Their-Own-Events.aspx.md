---
layout: post
title: Aggregate Roots Should Look After Their Own Events
category: Domain Driven Design
---

Today I've stumbled upon (well someone re-tweeted it) this [interesting post](http://mat-mcloughlin.net/2014/05/22/entities-should-look-after-their-own-events) . However I must disagree with the OP approach and this my reason why. **Update**: Disagreeing means I have another opinion, not that Mat's wrong. All these being implementation details, every dev chooses what (s)he thinks to be the best approach.

 While I understand that we're dealing with an example and I shouldn't take things literally, I couldn't help but notice that the Site aggregate root (I codesume) looks a lot like a container for other entities (it's a Site holding a collection of Account). This is maybe the biggest mistake when modelling an AR, to treat it as a container and in that example it does looks like it is one.

 But let's consider that the Site really is an AR and Account is an entity which partly defines the Site concept. We do care about AR as the most important concept of the aggregate. Therefore we're working just with the AR and in this case the repository will restore the AR from a stream of events (technically the entity restores itself but that's not the point here).

 The AR implementation details will actually decides which entity changes based on the event. I don't think that children of an AR should care about event sourcing because they're more of an implementation detail of the AR. Sure they might be entities, but that doesn't make them important enough in that context. The only star is the AR.

 And the AR isn't just a wrapper for its children objects. It should make sense for the Domain to actually implement a behaviour in an AR. Just as the Repository, we don't use it because we want a facade over other objects, the AR utility lies in the fact that it defines a (more) complex concept which makes uses of other minor, more generic domain concepts and encapsulates business interaction rules which happens to be excodessed as objects.

 That's why it matters a lot to get the correct AR design. If it's wrong then you'll only have a container with wrapper methods while the real behaviour will belong elsewhere. And you'll be keeping the AR class because Evans said that we should have one and the repositories work with one.

 Back to Mat's example, the Account is basically a minor domain concept defining the Site (it should be if the modelling is correct). When something changes (or the object is restored) one or more events are applied. By the Aggregate Root to the Aggregate Root.

 For me it makes no sense to make an implementation detail (like Account is for Site) handle their own event stream. After all, Event Sourcing (ES) is a technical solution, the event store cares only about a stream of events regardless to the object it belongs to and the repository and the outside 'world' works only with the AR. Why should we burden a lower level object to support ES?

 Of course, you want some things to be, let's say, not public. I mean maybe we want to check if an Account is Active, so it makes sense for it to have a public getter, but maybe not a public setter. Internal is a good fit here, not fool proof but at least it hints that it's not for everyone to change. Even better, an interface can be used if it doesn't complicate things.

 So, I think that ES makes sense only at the AR level. The rest of the objects used by the AR, because they make sense only in AR context (quite important!) are the AR's responsibility to persist. An on this note, this means that I consider the the AR should (it's not a hard rule after all) also generate all the relevant events. Let's not forget that the AR implements one domain concept with all the relevant behaviour. It just makes sense for it to be the domain events authority for that concept.

 I would model differently Mat's example. Site is an AR but not for Account which itself is (can be) an AR. And maybe Activated doesn't belong to Account as well. I mean an account is an account, activated or not. Activation is a status i.e a kind of flag to tell _other_ concepts/behaviour if/how they should use the Account. It's debatable if it makes sense to be part of the Account concept. I'm only guessing since I don't know exactly what Mat's Domain really is.

 In conclusion, I think of the AR as the main object of an aggregate, one which is defined by/coordinates/uses the "lesser" objects. Therefore, in my opinion, the AR should be the only one event sourced.


