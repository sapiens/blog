---
title: Aurelia With Razor Views And NancyFx
layout: post
category: Javascript
---

I've been toying these days with [aurelia js](http://aurelia.io) trying to do some tutorials and understand how I can integrate it with my current NancyFx app (I'll have several aurelia apps each dealing with a certain aspect of my project).  Each aurelia app has the sources (html views, razor view and view models written in typescript) in a directory where it makes sense (e.g: ~/Account/Settings) next to nancy modules, specific services etc (in case you're wondering, I'm using a business component based architecture where everything related to a component is in one directory, regardless of the technical aspects). The resulting static assets (js, maps, plain html) are copied to a `/content/[app name]` directory.


## The Problem

Aurelia views are, by convention, in the same place and have the same name as the view model in my case, in the `/content/[app]` directory. Not really an issue, however I need some views to be generated i.e to be cshtml files which are found elsewhere. I can configure aurelia to look for views not ending in html, but I don't want every view to be a Razor file. Most of the views are plain html and only some of them need processing.

Basically, I need a way to use both simple html files and Razor views with minimal fuss.

## Solution

After some thinking I came to the conclusion that it's better to implement everything server side. From aurelia's point of view every request is for a .html file. It's up to nancy detect which is which and to server either a static file or a processed razor view. By default, the `/content` directory is designated to contain only static assets so I needed a convention to filter the html files.

However, I need a way to mark that some html requests are for razor templates. Since every Razor view has a view model, I've decided to create an attribute which will mark a view model (and by convention the view) as an aurelia view.

```csharp
  public class AureliaAppAttribute:Attribute
    {
        public string AppName { get; set; }

        public AureliaAppAttribute(string appName)
        {
            AppName = appName;
        }
    }
```

As I've said, I have more than one aurelia app, so each view model needs to declare to which app it belongs. For the `~/Account/settings` app, any view model declared in that directory(and children of it) will be decorated with  `[AureliaApp("settings")]`.

Then I have an object which will recognize the aurelia view requests that are Razor views. I need that in order to filter the requests and allow the static html files to be served normally.

```csharp
public class AureliaRouter
   {
       public static bool IsAureliaViewModel(Type t)
       {
           return t.HasAttribute<AureliaAppAttribute>();
       }

       public static string ContentDir = "/content";

       Dictionary<string,Func<object>> _handlers=new Dictionary<string, Func<object>>();


       public bool IsAureliaView(string path)
       {
           return _handlers.ContainsKey(path);
       }

       public object GetModel(string path)
       {
           return _handlers.GetValueOrDefault(path,()=>null)();

       }

       public void AddView(Type viewModel)
       {
           var appName = viewModel.GetAttributeValue<AureliaAppAttribute, string>(a => a.AppName);
           var url = ContentDir + "/" + ExtractViewPath(viewModel.Namespace, appName) + "/" +
                     viewModel.Name.RemoveLastChars(5).ToLower()+".html";
           _handlers.Add(url,viewModel.CreateInstance);
       }

       static string ExtractViewPath(string nspace,string appName)
       {
           var idx = nspace.IndexOf(appName);

           return nspace.Substring(idx,nspace.Length-idx).Replace('.', '/');
       }
   }

//search and register the view models
public static void RegisterAureliaViews()
       {
           typeof (RoutingUtils).Assembly.GetPublicTypes(AureliaRouter.IsAureliaViewModel)
               .ForEach(t=>_aurelia.AddView(t));
       }

```

This object simply computes the url that would match the razor view, based on a view model. So for a view found in `~/Account/Settings/foo/bar.cshtml` it will generate the path `/content/settings/foo/bar.html` which is what aurelia will request. Once we have this we can filter the requests by setting our static content conventions in the nancy bootstrapper.

```csharp
          nancyConventions.StaticContentsConventions.Clear();
           nancyConventions.StaticContentsConventions.Add(StaticContentConventionBuilder.AddDirectory("content",null,"js","css",".map"));
           nancyConventions.StaticContentsConventions.Insert(0, (ctx, root) =>
           {
               var req = ctx.Request.Path;
               if (req.StartsWith("/content") && req.EndsWith("html"))
               {

                   if (RoutingUtils.Aurelia.IsAureliaView(req))
                   {
                       return null;
                   }

                     var file = Path.GetFullPath(Path.Combine(root, req.TrimStart('/')));
                   return new GenericFileResponse(file, ctx);
               }

               return null;

           });

```

Now, since I have no clue how I can return a razor View from the static content handler, I'll need a module which will handle the requests for the razor files. Here it is in all its glory.

```csharp

public class AureliaViewsModule:NancyModule
    {
        public AureliaViewsModule()
        {
            Get[@"/content/^([a-z\.\/]*\.html)$"] = _ =>
                View[RoutingUtils.Aurelia.GetModel(Request.Path)];

        }
    }

```

And one more thing, all razor views need to to have an empty layout, we want the template only.

If you're using Asp.Net Mvc, the approach is very similar, you can filter and return a proper view/static template from an action filter.
