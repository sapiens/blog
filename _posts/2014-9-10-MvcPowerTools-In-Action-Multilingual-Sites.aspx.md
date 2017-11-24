---
layout: post
title: MvcPowerTools In Action - Multilingual Sites
category: MvcPowerTools
---

Recently I've "developed" a simple brochure site which had the "catch" that it must be bilingual. Since it was mainly a static site (no db whatsoever) I needed to write the texts directly in html. Also, for SEO purposes the urls should change according to the selected language.

 Basically the plan was this: I have the layout in common with localized partial for meta description and each site page was a view per language and urls look like this: "mysite.com/home" , "mysite.com/accueil" . No language identifier in urls, the route for each url contains a 'lang' parameter. The contact page contained a form whose labels should be also localized.

 I had a _Translator_ class,nothing fancy, just a dictionary and some helpers. Let's start with the urls. Using routing conventions I came up with this convention:

  

```csharp
    public class MultilingualRouting:IBuildRoutes
    {
        private MultilingualTexts _translator;

        public MultilingualRouting()
        {
            _translator = Translator.Instance.Data;
        }

        public bool Match(ActionCall action)
        {
            return true;
        }

        public IEnumerable<Route> Build(RouteBuilderInfo info)
        {
            var texts = _translator.TranslationFor(info.ActionCall.GetActionName());
            return texts.Select(t =>
            {
                var route = info.CreateRoute(t.Value.ConvertAccentedString().MakeSlug());
                route.Defaults["lang"] = t.Key;
                return route;
            });

        }
    }


```
  The action name is the translation key. The translator returns a KeyValue array "language"=>"text". So, for each available language I generate a route having the localized (link) text converted as url without accents. Then the language is set for that route. All the routes contain the language associated with it.

 Next, the view engine conventions. The Views directory structure looks like this ![views directory](http://imgur.com/FlABmvO)

 And the conventions

  

```csharp
public class MultiViewEngine:BaseViewConvention
{
    public override bool Match(System.Web.Mvc.ControllerContext context, string viewName)
    {
        return true;
        return viewName.StartsWith("_");
    }

    public override string GetViewPath(System.Web.Mvc.ControllerContext controllerContext, string viewName)
    {
        return "~/Views/{0}/{1}.cshtml".ToFormat(controllerContext.RouteData.Values["lang"], viewName);
    }
}

public class DefaultEngine : BaseViewConvention
{
    public override string GetViewPath(System.Web.Mvc.ControllerContext controllerContext, string viewName)
    {
        return "~/Views/{0}.cshtml".ToFormat(viewName);
    }
}


```
  Not much to explain here. The current language is used as the directory where the view should be. If not, then it should be in the "~/Views" folder.

 Finally, the convention for our form labels

  

```csharp
 conventions.Labels.Always.Modify((tag, info) =>
        {
            var translator = Translator.Instance.Data.GetTranslations(info.ViewContext.HttpContext.CurrentLanguage());
            return tag.Text(translator[info.Name]);
        });


```
  By default, the label is the field name (the name of the model property) but with this modifier, we'll use it as a translation key to get the localized text then assign it to the label. Other approach would be generate directly the label with the localized text, but I just care about changing the text I don't want to re-write the whole label tag generation regardless how trivial it is. It's much simpler to just change the text.

 As a thumb rule, it's better to modify than to build (generate) if the tag already has a default, simple builder. For example, if I want all my text areas to be 7 rows, I'd write something like this

  

```csharp
 conventions.Editors
            .ForModelWithAttribute<DataTypeAttribute>()
            .Modify((tag, info) =>
            {
                var att = info.GetAttribute<DataTypeAttribute>();
                if (att.DataType != DataType.MultilineText) return tag;
                var input = tag.FirstInputTag().As<TextboxTag>().Attr("rows", 7);
                return tag;
            });


```
  What about generating links in html? I want to use this

  

```csharp
@(Html.LocalizedLinkTo(d => d.Contact()))


```
  And here's the helper code

  

```csharp
 public static HtmlTag LocalizedLinkTo(this HtmlHelper html,Expression<Action<DefaultController>> action,string text=null)
    {
        var name = action.GetMethodInfo().Name;
        var translations = html.GetTranslations();
        text = textjQuery15206183123854091122_1410368302646translations[name];
        return html.LinkTo("default", text, action:name,model:new{lang=translations.Language});
    }


```
  I have only one controller in this app (remember, it's a brochure site) so I only need the action name to use it as a translation key, then the current language is passed as a route value.

 And this is how you can use [MvcPowerTools](https://github.com/sapiens/MvcPowerTools/wiki) to build multilingual sites.


