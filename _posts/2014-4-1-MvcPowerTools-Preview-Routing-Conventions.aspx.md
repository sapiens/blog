---
layout: post
title: MvcPowerTools Preview - Routing Conventions
category: MvcPowerTools
---

MvcPowerTools (MPT) is a library aimed at the **experienced** asp.net mvc developer. It's meant to give you the power and flexibility when you find the standard asp.net mvc to be too rigid or just in your way. MPT it's deeply integrated with asp.net mvc (5+) and gives you alternate ways to accomplish things. It's heavily inspired from FubuMvc, so if you're familiar with Fubu, you'll find many similar things here.

 Today's post is about _Routing Conventions_. More exactly, MPT allows you to define your very own routing conventions. One thing to note is that it doesn't replace the standard conventions, you can use both the 'old' style and conventions at the same time, because all the conventions do is to allow you to define how the routes are generated for controllers matching a criteria. Once again, it's a way to extend what asp.net mvc already offers.

 So, you may not like to write all your routing by hand (for obvious reasons). Or you might want to use a certain routing approach for some controllers, some attribute base routing for others, namespace based hierarchy for some etc. Of course, you can do all this right now, but ... what if you need to change your routes based on the SEO flavour of the day or to accommodate internationalization  or versioning or.. you get the picture. Would you prefer to do it by hand, changing God knows how many routes or modifying tens or hundreds of controllers or you prefer to change a convention in ONE place and that's it?

 The point with routing conventions is that allows you to decouple the route generation from the controllers in a very maintainable manner. You can have as many conventions you need for all the edge cases. You don't have to force everything into a one way of doing things.

 Enough talk, let's see some code. Let's suppose I want a namespace based hierarchy for all controllers from the "Admin" namespace. And I want that all actions names starting with 'get' to be constrained as GET and what starts with "post" to be constrained as POST (yes, convention not attribute, less clutter). To keep things tidy, I'll define the convention inside a module (there are other ways)

  
```csharp
public class AdminConvention : RoutingConventionsModule
        {
            public override void Configure(RoutingConventions conventions)
            {
                conventions.
                    If(action => action.Controller.StripNamespaceAssemblyName().StartsWith("Admin"))
                    .Build(action =>
                    {
                        var url =
                            action.Controller.ToWebsiteRelativePath(GetType().Assembly)
                                .TrimStart('~')
                                .TrimStart('/')
                                .ToLowerInvariant();
                        var rt = action.CreateRoute(url);
                        //todo if you want to add params to url


                        if (action.Method.Name.StartsWith("get", StringComparison.OrdinalIgnoreCase))
                        {
                            rt.ConstrainToGet();
                        }
                        else
                            if (action.Method.Name.StartsWith("post", StringComparison.OrdinalIgnoreCase))
                            {
                                rt.ConstrainToPost();

                            }
                                                       
                        return new[] {rt};

                    });

            }
        }
```
  Next, in the routing config file (from App_Start)

  
```csharp
RoutingConventions.Configure(cfg =>
           {
               cfg
                   .RegisterControllers(Assembly.GetCallingAssembly())
                   .HomeIs<SomeController>(c => c.Index(null,"something"));
               
               cfg.LoadModule(new AdminConvention());
           });
```
  Now, just for fun, I want that all CamelCase controller names to be rewritten like "camel-case". I add a modifier.

  
```csharp
cfg.Always().Modify((r, action) =>
		   {
			   var con = action.GetControllerName();
			   var i = 0;
			   var final = new List<char>();
			   foreach (var ch in con)
			   {
				   if (char.IsUpper(ch))
				   {
					   if (i > 0)
					   {
						   final.Add('-');
					   }
				   }
				   final.Add(char.ToLower(ch));
				   i++;
			   }
			   r.Url = r.Url.Replace(con, new string(final.ToArray()));
		   });
```
  Now everytime I want to change something, I change it here. Anything else stays untouched. Btw, the register controllers part is required.  While this is only an example, you can create much more specific conventions each in its own class (SRP!).

 MPT comes with 2 predefined conventions: **[HandlerRouteConvention](http://lostechies.com/josharnold/2011/07/26/handlers-a-useful-fubumvc-convention/) ** and **OneModelInHandlerConvention**. Personally, I prefer the latter one as it's easier to maintain.

 OK, thats' it for now, but here's a taste of things to come: view engine conventions, html conventions, apply filters by conventions, command and query controllers. In the mean time you can check the [GitHub repo](https://github.com/sapiens/MvcPowerTools)


