---
layout: post
title: MvcPowerTools Preview - View Engine Conventions
category: MvcPowerTools
---

What I like the most from FubuMvc ([RIP](http://jeremydmiller.com/2014/04/03/im-throwing-in-the-towel-in-fubumvc/)) is the the paradigm of 'make the framework serve you, define your own conventions that suits your style'. Simply put, you decide the way you want to structure and code an app, not some framework. Frameworks are good but they shouldn't be in your way, you shouldn't force your app to comply with the framework, it should be the framework who should comply with your approach.

 Asp.Net Mvc is a good, easy to use framework but it still suffers from some 'you have to do things this way' symptoms. One example, why should I always inherit from Controller, instead of using any class and have the dependencies I want (HttpRequest, HttpResponse etc) injected? There's a lot of coupling in asp.net mvc, but this is how it is for now. I'm doing my part with MvcPowerTools to provide SOLID options for some features for when you reach the stage when you begin to fight the framework. I've already written about [Routing Conventions](http://www.sapiensworks.com/blog/post/2014/04/01/MvcPowerTools-Preview-Routing-Conventions.aspx) and [ActionFilters Conventions](http://www.sapiensworks.com/blog/post/2014/04/03/MvcPowerTools-Preview-Action-Filters-Conventions.aspx) , now it's time to tackle the view engines a bit.

 I'm not really a fan of "~/Views/[controller]/[action].cshtml" . It's a way to structure things but it gets tedious with many controllers and actions. Also throwing everything in Shared is not a good idea either. I like the 'Handler' approach for my controllers (i.e the controller is the action, its methods represent handlers for each supported http method) so the view name is the controller name. And I want that at least for a group of controllers to have the view in the same place as the controller. For other parts of the site, I want the views grouped into a theme/skin with no controller in sight. In a nutshell, I want to name and put my views wherever I need and sometimes that's not a predefined location (think themes, you want the user to copy the theme into directory with whatever name and the app should be able to use it).

 How to achieve that with the default aps.net mvc view engine? First of all, let me tell you a little trick: a view engine in asp.net mvc is actually a view locator. Then you have the templating engine (Razor for example) who does the real work. MS decided to couple these 2 so that's why there is an aspx view engine and a razor view engine. It should be one view engine (locator) supporting _n_ template engines, but I digress.

 Anyways, if you want to change the view location or even use a different view name convention (not the action name) you can do it by inheriting (yeah...) one provided view engine and then change the paths. What if you want to support themes i.e have another dynamic value (besides controller and action) in a path of a view? Well... you kinda have to rewrite a lot of what the view engine does, because it only provides 2 variables: controller and action name. What if you want (gasp!) different naming and path conventions for your views in the same application? Please, don't say:"Why would you want that? One convention to rule them all is the best". Yeah and one model for everything is also the best.

 Long story short, if you truly want to be in control of your views, there's a lot of work and some pain with asp.net mvc. Let MvcPowerTools (MPT) ease that pain with the FlexibleViewEngine. Yeah, it's flexible because YOU decide the conventions it uses. And you can have as many conventions as you need. You can start with one, the add some more. The only effort is to write the convention class. Nothing else.

 For this example, I want my views in the same folders with my controllers and the view has the controller name. Let's see how hard it is to implement it.

  
```csharp
public class ViewsPolicy:BaseViewConvention
    {
        public override bool Match(ControllerContext context, string viewName)
        {
            return !viewName.StartsWith("_");
        }

               public override string GetViewPath(ControllerContext controllerContext, string viewName)
        {
            var ctrl = controllerContext.Controller.GetType();

            var path = ctrl.ToWebsiteRelativePath(GetType().Assembly);
            return path + ".cshtml";
        }
    }

	//let's enable all conventions defined in the assembly
	 FlexibleViewEngine.Enable(c =>
            {
                c.Conventions.Clear();
                c.RegisterConventions(typeof (ConfigTask_ViewEngines).Assembly);
            },removeOtherEngines:true);
```
  Lots of complicated code here, let me help you understand it: first we decide when to apply the convention, in this case the view which isn't a partial (the underscore prefix designates a partial). Next, we need the path to the controller and we have a nice extension method (part of the tools) that takes a namespace and returns it as a path. We add the desired suffix and that's it.

 Now, let's see the code for when I want to have views with the controller name directly in ~/Views but with theme support.

  
```csharp
/// <summary>
    /// Used with Handler pattern controllers
    /// </summary>
    public class ViewIsControllerNameConvention:OneLayoutConvention
    {
        public override bool Match(ControllerContext context, string viewName)
        {
            return !viewName.StartsWith("_") && !FlexibleViewEngine.IsMvcTemplate(viewName);
        }

        public override string GetViewPath(ControllerContext controllerContext, string viewName)
        {
            var theme = controllerContext.HttpContext.GetThemeInfo();
            var controller = controllerContext.RouteData.GetRequiredString("controller");
            if (theme==null)
            {
                return "~/Views/{0}.cshtml".ToFormat(controller);
            }

            return "{0}/{1}.cshtml".ToFormat(theme.ViewsPath,controller);
        }
        
    }
```
  I think only one thing needs to be explained: _GetThemeInfo()_ . Theme support is provided by MPT via extension method and configurable structures. You only need a view engine capable to use them.

 In conclusion, you get the power to control the view engine and writing a convention is as hard as you've seen above. A word of advice, make separate conventions for views and partials, it's easier this way.

 P.S: I do understand if you don't find this feature useful, the point of MPT is to provide options for those who want them. Having alternatives is great, especially for those 1% edge cases asp.net mvc won't place nice with.


