---
layout: post
title: How To Use Razor with NancyFx
category: NancyFx
---

After using Nancy to build an Api app, I've decided to give it a try and use it instead of Asp.Net Mvc. I was a bit weary because I knew things are not that straightforward when it comes to Razor. Turns out I was partially right.

 But first things first. The [docs](https://github.com/NancyFx/Nancy/wiki/Razor-View-Engine) do a pretty good job explaining how to configure Razor, however there are some gotchas.

 1. Every bit of code you'll use in a razor view if it happens to be in a different assembly than the view, needs to be **manually** referenced, regardless if the current assembly reference it and your view has the required using statement. Coming from Asp.net Mvc you might be tempted to configure razor in app/web.config. Personally, I find it easier to use the programmatic option, that is implementing `IRazorConfig`. Now, everytime you need an assembly referenced in a Razor view just add it to that class.

 Here's an example

```csharp
 public class RazorConfig : IRazorConfiguration
   {
       public IEnumerable<string> GetAssemblyNames()
       {
           yield return "System.Web, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a";
           yield return "Common";

           yield return "HtmlTags";

       }

       public IEnumerable<string> GetDefaultNamespaces()
       {
        //contains html extensions
        yield return "Web._Auxilia.Html";

       }

       public bool AutoIncludeModelNamespace
       {
           get { return true; }
       }
   }

 ```

 While my webapp references System.Web and other assemblies, I still need to include it here for Razor to know about it. And this needs to be done when refactoring moves some code (used by the view) from one assembly to another or one namespace to another.

 2. All of your views need the `@inherits Nancy.ViewEngines.Razor.NancyRazorViewBase<My_Model>` with full path for the view base, instead of `@model MyModel` . This is easily solvable with a file template (at least if using Resharper).

 3. When defining a view convention the path **should not start** with '/' .

## View Conventions

 And speaking of view conventions, this is a very nice part of Nancy (you can get a similar feature in Asp.Net Mvc using my [MvcPowerTools library](https://github.com/sapiens/MvcPowerTools/wiki/ViewEngine)) that allows you to configure the view path you want. This means that you can put your some of your views in a View directory, others in next to the Nancy module or the view model itself. All these with just a couple lines of code.

 Here's an example

```csharp
 //in my nancy bootstrapper
protected override void ConfigureConventions(NancyConventions nancyConventions)
       {
           ConfigureViews(nancyConventions);
       }

       private static void ConfigureViews(NancyConventions nancyConventions)
       {
           nancyConventions.ViewLocationConventions.Clear();
        nancyConventions.ViewLocationConventions.Insert(0, (view, model, ctx) => "{1}/{0}".ToFormat(view, ctx.ModulePath));
           nancyConventions.ViewLocationConventions.Add((view, model, ctx) =>
           {
               var tp=(model.GetType() as Type);
               var asm=tp.Assembly.GetName().Name;
               var path = tp.Namespace.Substring(asm.Length+1).Replace('.','/');

               //path is model namespace without the assembly name
               return "{0}/{1}".ToFormat(path,view);
           });

           nancyConventions.ViewLocationConventions.Add((view, model, ctx) => "common/{0}".ToFormat(view));
       }

 ```

 Since I don't want to use any existing conventions, I clear the list and define my own. The first convention tells the engine to look for the view in a directory that has the same name as the module path. This usually happens if the module is used to define an equivalent to a asp.net mvc area. And usually, the module itself is in a directory with that name.

 The next convention is the most interesting one. It's for views sitting next to their view model. While in asp.net mvc returning a `View(model)` meant the view name is the action name found in a controller (or Shared) directory, in Nancy `return View[model]` looks for a view with the model type name (but without the "Model" suffix). And this might be strange but it's actually a great feature: just look at the view/model name and you already know the model/view name ;) . Also this means you should use one view model per view. And as a tip, **always** define a view model even if you plan to reuse an existing data structure. It's a very small price to pay for maintainability.

 As you can see there's no view extension mentioned. That's because the view locator searches for all files with that name regardless of their extensions and then uses the found extension to identify the view engine. This way, switching for Razor to Spark (for example) means you just have to install the Spark view engine and your code stays untouched.

## Html Tags and Helpers

 In this regard, you don't have much. While there is a nuget bringing the asp.net mvc html helpers to Nancy, it's pre release and it's a port. This is not wrong, but I do prefer a more fluent (read: not ugly) way of doing things. Besides writing the html code directly , which I don't recommend as a general thumb rule, you can use the [HtmlTags](https://github.com/darthfubumvc/htmltags) library to define your own Html Helpers for Nancy. It's more work but it's trivial and you get some cool benefits.

 Here's an example of a 'register user' form

```html
 <form action="/register" method="POST">

    @Html.HiddenTag(d=>d.OperationId).Raw()
    <div class="form-group">

        @Html.Label(d=>d.Fullname).Raw()
        @Html.Textbox(d=>d.Fullname).AddClass("form-control").Raw()
        @Html.ValidationText(d=>d.Fullname).Raw()

    </div>
    <div class="form-group">

        @Html.Label(d=>d.Email).Raw()
        @Html.Textbox(d=>d.Email).Raw()
        @Html.ValidationText(d=>d.Email).Raw()

    </div>
    @Html.FormField(d=>d.Password,()=>Html.Textbox(d=>d.Password).PasswordMode())

    <div class="form-group">
        <input type="checkbox" name="RememberMe" value="true"/><label>Remember Me</label>
    </div>

    <button type="submit">Go</button>

 </form>

 ```

 All the helpers are defined by me, they just provide a simple way to use HtmlTags in a Razor view. Here's a small excerpt

```csharp
      public static TextboxTag Textbox<T>(this HtmlHelpers<T> html,Expression<Func<T,object>> valueSelector)
       {
           var name = valueSelector.GetName();
           return html.Textbox(name);
       }

      public static TextboxTag Textbox<T>(this HtmlHelpers<T> html, string name)
      {
          var val = html.Model.GetPropertyValue(name) ?? "";
          return new TextboxTag(name, val.ToString()).IdFromName();
      }

       public static HiddenTag HiddenTag<T>(this HtmlHelpers<T> html, Expression<Func<T, object>> valueSelector)
       {
           var name = valueSelector.GetName();
           var result= new HiddenTag();
           result.Name(name).IdFromName();
           return result;
       }

      public static IHtmlString FormField<T>(this HtmlHelpers<T> html, Expression<Func<T, object>> valueSelector,
           Func<HtmlTag> factory = null)
       {
           if (factory == null)
           {
               factory = () => html.Textbox(valueSelector);
           }
           var result = new DivTag().AddClass("form-group");
           result
               .Append(html.Label(valueSelector))
               .Append(factory())
               .Append(html.ValidationText(valueSelector));
           return result.Raw();
       }

       public static HtmlTag ValidationText<T>(this HtmlHelpers<T> html, Expression<Func<T, object>> valueSelector)
       {
           var name = valueSelector.GetName();
           return html.ValidationText(name);
       }
       public static HtmlTag ValidationText<T>(this HtmlHelpers<T> html, string name)
       {
           var result = html.RenderContext.Context.ModelValidationResult;
           if (result.IsValid) return HtmlTag.Empty();
           var error = result.Errors.GetValueOrDefault(name, new ModelValidationError[0]).FirstOrDefault();
           if (error==null) return HtmlTag.Empty();
           var tag=new SpanTag();
           tag.Text(error.ErrorMessage);
           return tag.AddClass("error");
       }

       public static IHtmlString Raw(this HtmlTag tag)
        {
            return new NonEncodedHtmlString(tag.ToString());
        }

 ```

Some interesting things:

* All (almost) helpers return a `HtmlTag` object or a type inheriting `HtmlTag`
* Because Razor html encodes everything, we need to tell it to not encode the tag, hence the `Raw()` extension method
* Validation result object (created by a call to `Validate` or `BindAndValidate` in a Nancy module) can be found at `[html helper].RenderContext.Context.ModelValidationResult` . That's where the `ValidationText` takes the errors messages from.

Creating your own helpers means it's easier to add changes later. For example, I can decide to create a helper `AsSubmit()` for a button or link tag, which can add or remove a specific css class or attribute or include directly a css class for a specific tag (such as "form-control" for textboxes). HtmlTags library has some support for defining your own html conventions (that can manipulate tag generation based on model) but the docs are lacking in this regard, AFAIK is still work in progress.


## Conclusion

 Clearly, using Razor with Nancy is not as smooth as with Asp.Net Mvc, but for me it's not a big deal. I like Nancy as a framework and the 'price' for using razor is quite small once you have a workflow defined. Actually, I'd say it's easier for a asp.net veteran to write a maintainable app with Nancy, while it's easier for a beginner to use Asp.Net Mvc. You can do everything with both, but with Nancy **you** are more in control.
