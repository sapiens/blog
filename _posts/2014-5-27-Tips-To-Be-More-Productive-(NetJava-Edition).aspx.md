---
layout: post
title: Tips To Be More Productive (.Net/Java Edition)
category: Best Practices
---


## Automate what you can

 In a asp.net mvc multi tenant app I needed to set the tenant id codetty much everywhere. Yes, it's only one line of code but I have to repeat it everywhere and I want to "forget" about it.   
 So, I've decided to use an action filter for that. Now, the framework is executing that line automatically.

 
## Code reusable behaviours against interfaces

 I'd say abstractions but then you might think an abstract class is good too. May be, it depends, but using interfaces allows for more granular behaviour and less coupling. And please, try to avoid using inheritance (a common base class) for reusability purposes. You should keep inheritance for _is a_ sub types.

 Remember our codevious multi tenant scenario? I need to set the tenant id. However, I have many unrelated input models (which will be used by an application service to do the actual job) and the relevant property for this use case is Tenant. So, I did this

  
```csharp
public interface IHaveTenantId
 {
     TenantId Tenant {get;set;}
 }
```
  And made any input model to implement that interface. The filter would check if the model is an IHaveTenantId and set it if not null. The filter knows only about the interface it doesn't care about a concrete type. The filter automatically works with ANY model as long as it implements _IHaveTenantId_.

 Another example involves extension methods. Hint: what's relevant here is that we have a behaviour (regardless where it's implemented) for a property with can be part of many different objects. Here's how code used to do mapping for a property reused by many business objects looks like

  
```csharp
public interface IHaveBusinessContent
  {
      BusinesText Content {get;}
  }
  
  public interface IHaveSimpleContent
  {
     string Content {get;set;}
  }
  
  public static Map(this IHaveBusinessContent source, IHaveSimpleContent dest)
  {
   //do mapping
  }
```
  It doesn't matter which objects the source or dest are. The behaviour implemented in the extension method works for _ANY_ object implementing the specified interfaces. And I know about Automapper, but sometimes you need more logic for mapping and here mapping is just an example, this approach can be used for any reusable behaviour. Now, what if out model needs a Tenant and has a BusinessText Content? Just invoke the already defined methods.

 A behaviour in order to be reusable has to be specific enough (granular). We want the implementation to be as decoupled as possible from a concrete type and that's why we 'couple' it to an interface (it's an application of the Interface Segregation principle, the I from SOLID). Note that these kind of reusable behaviour are usually infrastructural concerns. Reusing business logic doesn't really make sense outside a specific business context.

 
## TDD like there's no tomorrow

 If you're lazy, like me, you'll love TDD. First of all, you don't have to think about all cases and possible use cases that may or may not happen. You get to focus on one thing at the time. And you write a test which excodess an expectation. And then make that test pass. Then do the same for all the expectations you have from that object. Only what you know you need at that moment. Nothing more.

 You get to work less. Sure, you are writing tests which means you write additional code, but you're actually accomplishing many things at once (productivity): you code ONLY the behaviour that is needed and the code will work as expected (and that's not an assumption, you have passing tests proving this). No time wasted dreaming future scenarios and much less time spent hunting bugs. Every experienced developer can tell you stories about how they wasted hours or even days to hunt down whatever bug deeply burrowed in a jungle of objects spanning multiple layers. But with tests you already know what works and it's simply easier to locate the problem.

 Productivity increases because you're spending less time to deliver reliable high quality code, which in turn allows you to change things faster with less unwanted side effects (bugs). Btw, TDD is only a tool, it won't think the code in your place. You, the developer, are the designer and you can still write crappy code using TDD i.e TDD helps but it's not a silver bullet, you still need to be a good developer.

 
## Use a DI Container

 Setting up a DI Container is among the first thing I do when I begin working on a non-throw away app (prototypes, experiments). It takes only 1-2 minutes and allows me to just add classes, without wasting time to do manual wiring. I 'win' maybe 3-4 seconds but the main benefit is that I'm just defining types and everything works. From time to time, I have to configure a new container convention or adjust an existing one but that doesn't happen very often.

 And I love that I can just define an interface, then an implementation (sometimes more than one) and all objects taking that interface as a constructor dependency just work. That's the beauty of a DI Container. Adding new types means you can automatically use them.

 I know some devs disagree with this. They consider that a DI Container is overkill unless you have objects with a lot of nested dependencies. But, as I've said before, I'm lazy and I want the tool to work for me even in more simple scenarios. The cost of using a DI Container is so low (from a productivity point of view), IMO, that I hardly consider it.

 You can save time bit by bit (seconds at the time) but those seconds add up and best of all, those seconds can be used for other (creative) things than for repetitive/boring tasks. I mean, why manually writing an object factory instead of configuring it once or a few times and let the factory pick up the new types on the fly? We need to simplify our work as much as possible.

 
## Use a (micro)ORM

 Valid only with a RDBMS, but nevertheless RDBMS are and they will be used for a long time. Truth is, you don't have a valid excuse to use Ado.Net directly (except if you're building a data access tool). Doing things manually is boring and prone to error and the "this the way to get fast queries" argument is not valid anymore. A full 'heavy' ORM is slower but it has other features and the app might benefit more from them (working with the db can be easier and not all the app really need absolute performance). If you want very fast queries you do have a plethora of micro ORMs to choose from (I recommend my very own [SqlFu](https://github.com/sapiens/SqlFu), of course). They do the boring stuff for you at around 95% speed of a manual Ado.Net solution. I can guarantee you won't feel the few ms per 1000 queries but you'll feel the annoyance of doing the ado.net dance every time.

 Use a micro ORM to save time/sanity . You'll still be tweaking sql by hand if that's your thing. But without the boring bits.

 
## Use Open Source Software

 I know some shops/companies who say "We don't use external, 3rd party libs" where external means "not Microsoft". You should avoid them. There's a lot of valuable software in the Open Source world and while I understand a business wants support and trusts more a company which sells their software, the reality is even Microsoft has opened sourced a lot of things. It makes little sense not to use existing libraries or coding functionality just because they're free open source.

 Sure, most of the time you won't have formal support (although I can bet a lot of OSS authors wouldn't mind help you for a fee) and documentation might not be the best, but there are products used by hundreds of thousands of developers from all over the world. If it works for them, chances are it might work for you.And in many cases is much easier to tweak an existing solution rather than coding it from scratch. If you want to be more productive you should use all the tools that make you be more productive.

 One more thing, writing high quality code (reliable, maintainable) is not bullshit, they're not just buzzwords. Good design and decoupled code allows you to get a better product faster, we don't want to write good code because we're idealists. This is pragmatism, you want to work less, deliver good products, spend more time doing more fun stuff than fixing bugs or rewriting 1/3 of an app because of a new feature.

 You want to write good code for YOU, because YOU'll be maintaining it. You want to work more or less?


