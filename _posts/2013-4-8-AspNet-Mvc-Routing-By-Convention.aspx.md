---
layout: post
title: Asp.Net Mvc Routing By Convention
category: Caveman Tools
---

My brief [adventure with FubuMvc](http://www.sapiensworks.com/blog/post/2013/01/02/My-FubuMvc-Adventure-First-Steps.aspx) had a greater impact on me than I expected. I happily adopted some of its features to use with Asp.Net Mvc. One thing that I liked a lot was configuration by convention. And that didn't mean: "Here's the convention, now use it" but the ability to define your very own conventions. This way everybody can do things how they want and everyone is happy. Win-win.

 I adopted the [handler convention](http://lostechies.com/josharnold/2011/07/26/handlers-a-useful-fubumvc-convention/) as it cleans up the code a bit, but I needed a way to define the routes. Writing them by hand is not an option and using generic routes like "{controller}/{action}" means a lot of fun debugging them.

 I know about [Attribute Routing](http://attributerouting.net/) but decorating each action or controller every time is not a good solution for me. I mean there is only 1 attribute, but one for routing, another one for authorization, another one for theming etc and you have to remember a small stack of attribute to put on top of every controller.

 What I need is a way to define some rules that can be applied automatically. For example I wanted to:

  
  * generate the routes from the actions and use the naming convention to format the url. So having the controller "Pages" action "GetBy_Id_Page" would generate the url "pages/{id}/{page}". All POST actions would be simply called "Post" or should start with that prefix. 
  * every url must be prefixed with the "{tenant}" parameter and the route should contain the default value for it. 
  * every Admin action should contain 'Admin' in the url or even better, the urls should be namespace based i.e the url hierarchy should respect the namespace hierarchy.  Taking a hint from Fubu, I came up with a simple way to define routing policies. I considered 3 cases:  
- Generate 1 or more routes from an action  
- Format url according to a criteria  
- Apply a general policy for all routes

 This system is just an alternative way to define routing and it doesn't interfere with the manual route definition or any other way. It only provides a way to use conventions in order to define the routes.

 So, for my project I ended up defining these routing policies:

  
  * **HandlerRouteConvention** ([source here](https://bitbucket.org/sapiensworks/caveman-tools/src/46938e46afa956fcf9e76a5e6b0eb9a4f62920ac/src/CavemanTools.MVC/Routing/HandlerRouteConvention.cs?at=devel)) 
  * ****NamespacePrefixedUrls  
    ****  
```csharp
public class NamespacePrefixedUrls:IRouteUrlFormatPolicy
    {
        public bool Match(ActionCall action)
        {
            return true;
        }

        public string Format(string url, ActionCall actionInfo)
        {
            var prefix = actionInfo.Settings.StripNamespaceRoot(actionInfo.Controller.Namespace);
            prefix = prefix.Replace('.', '/').TrimStart('/');
            if (prefix.IsNullOrEmpty())
            {
                return url;
            }
            return prefix.ToLowerInvariant() + "/" + url;
        }
    }
```
  **** 
  * **EveryRouteHasTenantPrefix** **  
    **  
```csharp
public class EveryRouteHasTenantPrefix : IRouteGlobalPolicy
    {

        public void ApplyTo(Route route)
        {
            route.Url = "{tenant}/" + route.Url;
            route.Defaults["tenant"] = "main";
        }
    }
```
    The entire code to define routes using policies

  
```csharp
var routes = RouteTable.Routes;
    routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
     
    var policy = new RoutingPolicy();
    policy.RegisterControllers(typeof(Routing).Assembly);
    policy.GlobalPolicies.Add(new EveryRouteHasTenantPrefix());
    policy.UrlFormatPolicies.Add(new NamespacePrefixedUrls());
    policy.RegisterHandlerConvention();
    policy.Apply(routes);
```
  If I know I'll be needing more policies over time I can harness the power of the DI Container

  
```csharp
//register any convention with the DI Container (autofac)
  builder.RegisterAssemblyTypes(ThisAssembly).Where(t => t.DerivesFrom<IRouteConvention>()).AsSelf();
  builder.RegisterAssemblyTypes(ThisAssembly).Where(t => t.DerivesFrom<IRouteUrlFormatPolicy>()).AsSelf();
  builder.RegisterAssemblyTypes(ThisAssembly).Where(t => t.DerivesFrom<IGlobalRoutePolicy>()).AsSelf();            
 
 
 //tell RoutingPolicy to scan and register any policy it finds
  var policy = new RoutingPolicy();
  policy.RegisterControllers(typeof(Routing).Assembly);
  policy.RegisterPolicies(typeof(Routing).Assembly);
  policy.Apply(routes);
```
  Now, the only thing left is to write the policies and they will be used automatically. You can check out the [source code for the 'routing by convention' system here](https://bitbucket.org/sapiensworks/caveman-tools/src/46938e46afa956fcf9e76a5e6b0eb9a4f62920ac/src/CavemanTools.MVC/Routing?at=devel).


