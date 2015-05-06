---
layout: post
title: MvcPowerTools Preview: Action Filters Conventions
category: MvcPowerTools
---

Action filters are a great way to implement some aspect oriented programming and a powerful feature of Asp.Net Mvc. However, in a big application, it can become cumbersome to decorate controllers and methods with every filter you want. You can use global filters but those are .... global, they apply to everything. If you don't want to copy paste annotations every time, a workaround is to define a base class and make the controller inherit from it. But this is a workaround and on the long run, the solution isn't very clean at all. What if you need filters that are codesent in 2 base classes? Create a third with the common filters? Slowly the ball of mud gets bigger.

 [MPT](https://github.com/sapiens/MvcPowerTools) gives you the tools to implement this functionality without inheritance or other duct tape solutions. All clean and SOLID. And easy to use. The point is you define conventions based on which filters are applied to actions. Plain and simple. No tower of annotations required.

 As an example, I want that all my controllers from the Admin namespace to require authorization i.e use _[Authorize]_ . Also, I want that all actions names codefixed with 'ajax' to accept only ajax requests. And all the POST actions (either codefixed with 'post' or having [HttpPost]) should be the subject of _[ValidModelOnly]_.

  
```csharp
public class MyFilterConventions : FiltersConventionsModule
        {
            public override void Configure(IConfigureFilters filters)
            {
                filters
                    .If(a => a.DeclaringType.StripNamespaceAssemblyName().StartsWith("Admin"))
                    .Use<AuthorizeAttribute>()
                    .If(a=>a.Name.StartsWith("ajax"))
                    .Use<AjaxRequestAttribute>() //part of MvcPowerTools
                    .If(a=>a.Name.StartsWith("post") || a.HasCustomAttribute<HttpPostAttribute>())
                    .Use(()=> new ValidModelOnlyAttribute()) //part of MvcPowerTools
                    ;

            }
        }
		
 FiltersConventions.Config.RegisterControllers(Assembly.GetCallingAssembly());
 FiltersConventions.Config.LoadModule(new MyFilterConventions());
 FiltersConventions.Config.BuildAndEnable();
```
  That's all! Easier and clearly more maintainable than using inheritance, right? And your controllers stay clean. Now, one might argue that it is a good thing to have those annotations right on top of the action. Indeed, it's a visual help. However once you have more than 2 annotations, it becomes a clutter. In my opinion, it's much more clean to set up some conventions and respect those. But it's a matter of taste and this is an alternative to the 'standard' asp.net mvc approach. You can use both at the same time, it's a valid choice also.

 A note, if using a DI Container. If you're not creating the action filter instance yourself (like in the final convention above), you need to make sure the DI Container knows how to instantiate the filters you want.


