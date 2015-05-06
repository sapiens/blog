---
layout: post
title: DRY Code, Rich Code
category: Best Practices
---

The good part is that even junior devs know about DRY (Don't Repeat Yourself). The bad news is that a lot of devs believe that DRY is about avoiding repeating lines of code or properties names.

 But just like the Law of Demeter isn't about [counting dots](http://haacked.com/archive/2009/07/14/law-of-demeter-dot-counting.aspx/) DRY isn't about avoiding having 2 classes with the same property. It's all about context and behaviour.

 It's obvious that when you need the same behaviour implemented by a code block, you should refactor it to a function. This allows you to change the behaviour in one place only. This is the meaning of DRY, don't duplicate code which does the same thing. Reuse it. However...

 There is the issue of context. All model and behaviour happens within a context. You are structuring your app in layers(tiers) because you want each layer to handle one specific concern. Thus, the model for Persistence deals with databases, in Domain it's only about business concepts and use cases, in UI we care about User Interface and not about business logic or databases etc.

 So there are multiple models, each specific to its own context (layer, tier,component, module, subdomain etc), minding only their specific concern. But those models can't be that different, after all there are just different aspects of the same things. And in simple apps you see that the view model is 99% identical to the business data structures and 99% identical to the persistence model.

 And you think: "DRY!" and try to reuse it. One model to rule them all. But this will be a problem on the long run, because now, you have one model reused for different purposes. This is not DRY, this is a confusing mess, hard to maintain code.

 Reusability is great and desired, but the separation of concerns should be the primary criteria when deciding if something is indeed a repeat or it can be reused. This means you have to be aware at any time in what context you're working on and you have to maintain the discipline to treat each model in that context.

 It's normal human behaviour to look out for shortcuts, but if you want a highly maintainable application, don't use the same model outside its own context. If it's identical to other models, just define a new class inheriting the other or copy/paste directly the properties (not the methods though). The point is that the code in one context should be 'coupled' only to the model from that context (as much as possible) or a compatible context (part of the same responsibility).

 So don't reuse your business data structures as view models or as persistence entities. They serve different purposes and it's just a coincidence they look the same. Treat them like the separate models they are. It's important from an architectural point of view to maintain this separation, and the implementation details are yours to decide. And yes, copy/paste is a valid method and works great, but remember it's just a tool that can be easily abused.

 In conclusion, DRY doesn't mean "make a mess under the illusion of reusability", it's just a simple principle that reminds you to keep relevant behaviour in one place.


