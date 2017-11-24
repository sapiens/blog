---
layout: post
title: MvcPowerTools 1.0. Released
category: MvcPowerTools
---

After 6 months of work, I'm proud to announce to the Asp.Net Mvc community that MvcPowerTools (MvcPT) , aka FubuMvc goodness for AspNet Mvc, is finally released. Aimed at the seasoned developer, MvcPT brings the power of conventions to the asp.net mvc platform. What's the big deal about these conventions, you ask? Well, grab a be... chair and pay attention for 3 minutes.

 Imagine that you want to write maintainable code. How do you define maintainability in a few words? "Easy to change" . Simply put, you want the code to be changed easily and that's why you want to write decoupled code, prefer composition over inheritance, use an event driven architecture, respect SOLID principles etc. Using conventions is yet another tool in achieving maintainability. And not only that, it's about control. Do you want to force your app into a framework's constraints or do you want to make the framework adapt to your style? Who's in control, you or the framework?

 Using conventions means you decide the rules that a framework uses to provide a behaviour. Take for example, routing. If the routes are generated based on conventions (rules) you can come up with any convention you want that will create one or more routes based on a controller/action. Want to use attributes? OK! Want namespace based routes? Ok! Want to mix'n'match things based on whatever criteria? You get to do that! And best all, want to _change_ all things at once? Just change the relevant conventions.

 Let's talk about html conventions i.e rules to create/modify html widgets. When using the power of conventions, you can make bulk changes, without touching anything BUT the convention itself. Think about forms. In a big business app you may have hundreds of forms. If the form fields are generated using html conventions i.e the code looks like "@Html.EditModel()", then making ALL of the fields from all the forms use boostrap css amounts to define a single convention (2-3 lines of code). Change that convention and all hundreds of fields automatically change. It's simply DRY.

 Some devs don't like conventions because it does look like magic, I suppose it depends on the developer if they want _to control_ or _fear_ the magic. I do find useful, for example, to see the [Authorize] on top of a controller, but I get tired very quickly to write that attribute anywhere I need it and inheriting a AuthorizedController base class is just a bad solution. I just want to say directly: "All the controllers from Admin need to be authorized" or "All the controller BUT these [] need to be authorized" or "All my WebApi controllers respond ONLY to Ajax requests".

 Conventions allows you to do **mass control** as opposed to **micro managing** things. Tired of repeating yourself? DRY! Use a convention. Want to make bulk changes with minimum effort? Use conventions!

 I'm not cheering for a new golden hammer, but for another tool that will help us write maintainable code. And if you liked Jimmy Bogard's [put your controllers on a diet approach](http://lostechies.com/jimmybogard/2013/10/10/put-your-controllers-on-a-diet-redux/) you'll find a nice surprise built in MvcPowerTools.

 Now, go [read the docs](https://github.com/sapiens/MvcPowerTools/wiki), get the [nuget](https://www.nuget.org/packages/MvcPowerTools/) and start writing more maintainable apps!


