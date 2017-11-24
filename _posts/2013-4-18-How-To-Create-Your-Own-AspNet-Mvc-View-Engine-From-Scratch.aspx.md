---
layout: post
title: How To Create Your Own Asp.Net Mvc View Engine From Scratch
category: Caveman Tools
---

There are a number of reasons you might want to create your own view engine for Asp.Net Mvc. Mine was that I wanted to have finer control over configuration. You can do a lot with the default view engine but you have to use the golden hammer of inheritance. And for some 'advanced' features like theming you need to modify it (how lucky that asp.net mvc is open source).

 Anyway, regardless the reasons why you want to create your own view engine, here's how to build one from scratch. **Warning:** Code heavy post.

 In Asp.Net MVc the view engine has 2 concerns: to locate a view and to instantiate a view. Note that it has **nothing to do** with the template parsing and rendering so the same view engine can be used to render Razor, Webforms, Spark etc. It's important to understand the difference between the view engine and the template engine.

 Here, I want to create a flexible view engine, that is, it can be easily configured via conventions and can support any template engine.

  
```csharp
public class FlexibleViewEngine:IViewEngine
{
    public ViewEngineResult FindPartialView(ControllerContext controllerContext, string partialViewName, bool useCache)
    {    
    }
    
     public ViewEngineResult FindView(ControllerContext controllerContext, string viewName, string masterName, bool useCache)
    {
    }
    
    public void ReleaseView(ControllerContext controllerContext, IView view)
    {
    }
}
```
  These are the methods required by **IViewEngine**. You can see the **ViewEngineResult** which will be returned in both success and failure scenario. The difference is the when failed, it will contain a collection of searched view path, the ones you see in the yellow screen of death when asp.net mvc can't find a view.

 I've said above that this view engine will know how to work with any template engine and this means that it will require **view factories** injected in it. For easy config, I've decided to have a **FlexibleViewEngineSettings** object where factories can be added.

  
```csharp
public class FlexibleViewEngineSettings
    {
       
        public List<IViewFactory> ViewFactories { get; private set; }

      
        public List<IFindViewConvention> Conventions { get; private set; }
        
    }
```
  The **Conventions** list holds the conventions used to locate views.

 The view engine accepts an action that can be used to add view factories and view locator conventions.

  
```csharp
private FlexibleViewEngineSettings _settings=new FlexibleViewEngineSettings();

    //constructor
    public FlexibleViewEngine(Action<FlexibleViewEngineSettings> config=null)
    {
        if (config != null)
        {
            config(_settings);
        }
        
    }
```
  How does a convention looks like?

  
```csharp
public interface IFindViewConvention
    {
        bool Match(ControllerContext context, string viewName);
        string GetViewPath(ControllerContext controllerContext, string viewName);
        string GetMasterPath(ControllerContext controllerContext, string masterName);
    }
```
  Each concrete convention must decide when to apply and how to create a path for a given view or layout. One example can be the case of an Ajax heavy site that still wants to be indexed by search engines. When a search crawler is detected, a special SEO tailored view can be used.

 A view factory interface looks like this

  
```csharp
public interface IViewFactory
    {
        bool IsSuitableFor(string fileExtension);
        IView Create(ViewCreationData settings);
        IView CreatePartial(ViewCreationData settings);
    }
```
  The concrete factory will be used only for view paths ending with a suitable extension. This means that conventions (the view path generator) and factories work together. Although a partial view is a view, it's necessary to have separate methods because a partial view has special needs, for example, in razor case it should ignore **_ViewStartPage** .

 Ok, so now we have conventions and view factories in our settings. Let's see how they are used by the view engine.

 The main idea is that for a given view it processes all the conventions and uses the first which returns a valid view path. It then uses that path to search for a view factory which can handle the file extension. After that it asks the factory to create a **IView** instance which is returned via the **ViewEngineResult** object.

 If a returned view path isn't valid, it's added to a list of searched paths. If no conventions returned a valid path or there are no view factories which can handle it, an 'error' **ViewEngineResult** is returned containing the list of searched paths. That's all, pretty easy.

 And once the view engine is built along with some conventions and view factories, just hook it up

  
```csharp
ViewEngines.Engines.Clear();
    ViewEngines.Engines.Add(new FlexibleViewEngine(v =>
        {
            v.Conventions.Add(new MyViewConvention());
            //another view factory
            v.ViewFactories.Add(new MyViewTemplateFactory());
        }));
```
  I've included a Razor view factory and the Asp.Net Mvc view conventions (cshtml only and with theme support) by default, so this configuration is for only when you want to use your own conventions or another template engine.

 Here's an example of how to write a view factory

  
```csharp
public class RazorViewFactory:IViewFactory
    {
        private string[] _extensions = new []{"cshtml","vbhtml"};
        public bool IsSuitableFor(string fileExtension)
        {
            return _extensions.Contains(fileExtension);
        }

        public IView Create(ViewCreationData settings)
        {
            return new RazorView(settings.Context, settings.ViewPath,settings.MasterPath, true,_extensions);
        }

        public IView CreatePartial(ViewCreationData settings)
        {
            return new RazorView(settings.Context, settings.ViewPath, null, false, _extensions);
        }
    }
```
  That's all! You can check the entire [FlexbileViewEngine code here](https://bitbucket.org/sapiensworks/caveman-tools/src/688fd50964abfebfa979585efcb40e03e6a0262f/src/CavemanTools.MVC/ViewEngines?at=devel)


