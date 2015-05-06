---
layout: post
title: Domain Events Toolkit Released And My Git Experience
category: None
---

So, I decided that my Domain Events Framework was a bit to dependent on Rx, not only as a library but also as a mindset. The DomainEvents pattern is codetty straightforward and I thought that a simplified and lighter alternative is better. I mean you don't have to understand the Rx principles in order to use it. Rx is an amazing library but way to overkill for this.

 After a bit of thinking,I decided that is also better to come up with a different name, rather than change the version and break codetty much everything. Besides, 'Toolkit' is a better description, it's something small and quick. 'Framework' usually makes you think of something big, complex, clunky.

 A quick tutorial

  
```csharp
//define an event
public class OrderPlacedEvent:DomainEventBase<Order>
{
    public OrderPlacedEvent(Order data) : base(data)
   {
   }
 }

//define a handler
 public class OrderPlacedEventHandler:DomainEventHandlerBase<OrderPlacedEvent>
  {
    public override void Handle(OrderPlacedEvent ev)
    {
      //handle event
    }
  }

//create the events manager
static IApplicationDomainEventsManager DomainEvents=new ApplicationDomainEventsManager()

//register a handler for the event
var subscription=DomainEvents.RegisterHandler(new OrderedPlacedEventHandler());

//publish events
DomainEvents.Publish(new OrderPlacedEvent(myOrder))

//unregister the handler
subscription.Dispose();
```
  Pretty simple, heh? But you can find more details [on Github](https://github.com/sapiens/DomainEventsToolkit/wiki/Usage) .

 Speaking of Github, this is also the day I joined the cool kids and made an account on github and actually using Git. My experience was codetty much ugly. Yes, Git is known as being hard to use on Windows and it lives up to the expectations. The Github site has more features than BitBucket but also I found it more cumbersome to write the docs. I write Markdown every day on Stackoverflow so I'm used to the syntax, but it was painful to write it on Github.

 I codefer Mercurial every day (I learned it quite fast), but unfortunately Github is all the rage these days and Github uses only Git. Well more knowledge means more power after all.


