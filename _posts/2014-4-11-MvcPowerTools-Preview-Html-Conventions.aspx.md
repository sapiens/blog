---
layout: post
title: MvcPowerTools Preview: Html Conventions
category: MvcPowerTools
---

The moment we'll all be waiting for. Well, at least I was... I know this is (was) a Fubu feature that people used with asp.net mvc, but it was simply hat a developer extracted from Fubu then glued with the asp.net mvc. Html Conventions in [MvcPowerTools (MvcPT)](https://github.com/sapiens/MvcPowerTools) are a first class citizen, deeply integrated with Asp.Net Mvc. So, now you can use HtmlConventions like in Fubu (well the syntax resemble but it's not 100% identical) as a 'native' asp.net mvc feature.

 If you don't know what's the deal with html conventions, let me make you a very short codesentation. .Net Mvc templated helpers are great but they're a bit cumbersome if you just want something more of a widget, a mini-template that can be customized in more than 1 step. And sadly the mvc's behaviour is an all or nothing affair: you either use display templates for all properties in your model and you get away with _Html.EditorForModel_ or you have to write manually for every property _Html.EditorFor(m=>m.Property)_ just to have control for one property you want to be rendered differently.

 The point of html conventions is to define html rendering 'rules' that will apply to all model matching a criteria. This means you can make bulk changes just changing a few lines of code in a convention. As I've said before, it works at a **widget** level. It's possible but not recommended to generate ALL the html via conventions. Among the best use cases are form fields (input controls). If you're dealing with many forms and fields, html conventions are a productivity saver. Add/change a convention and all targeted controls are automatically updated. A note about testing, the deep integration with asp.net mvc is both a blessing and a curse, in order to test the conventions you have to mock at least ViewContext (which requires a ControllerContext and so on). For now it's like this, maybe in a future version I can avoid this tight coupling (I wonder if 'integrated' and 'decoupled' are compatible).

 In this post I'll show you some interesting examples to show you the possibilities, it's not really meant to be a tutorial. Let's begin!

 I happen to use angularjs for my mvc javscript needs and I'm using it when editing a form. So I want that every model that I annotate with [NgModel] to generate html inputs supporting anuglarjs i.e having the _ng-model_ and _ng-init_ attributes set. To define the conventions I recommend the following directory structure (each .cs file is a HtmlConventionsModule)

  
```csharp
App_Start - Html - Display - Build.cs
				          - Modify.cs
					  - Ignore.cs		
							
			   - Editors - Build.cs
			                 - Modify.cs
					 - Ignore.cs					
							
			   - Labels  - Build.cs
				          - Modify.cs
					  - Ignore.cs					
                  

				  
     public class NgModelAttribute:Attribute
    {
         
    }
	
	//namespace App_Start.Html.Editors
	
	 public class Modify:HtmlConventionsModule
    {
       
        public override void Configure(HtmlConventionsManager conventions)
        {
            var e = conventions.Editors;
			
			 e.If(m =>
                (!m.IsRootModel && m.ParentType.HasCustomAttribute<NgModelAttribute>() &&
                 !m.HasAttribute<NoAngularAttribute>()) || m.HasAttribute<NgModelAttribute>()
                )
                .Modify((tag, m) =>
                {
                    var input = tag.FirstInputTag();
                    if (input == null) return tag;
                    var ngName = "res." + input.Attr("name").ToLower();

                    input.Attr("ng-model", ngName);

                    if (!m.ValueAsString.IsNullOrEmpty() && !m.Type.IsUserDefinedClass())
                    {
                        if (m.Type.Is<bool>())
                        {
                            input.NgInit("{0}={1}".ToFormat(ngName, m.ValueAsString.ToLower()));
                        }
                        else
                        {
                            input.NgInit("{0}='{1}'".ToFormat(ngName, m.ValueAsString.AddSlashes()));
                        }
                    }
                    return tag;
                });
				
			e.Always.Modify((tag, i) =>
            {
                var input = tag.FirstInputTag(t => t.IsHiddenInput());
                if (input == null) return tag;
                if (input.HasAttr("ng-model"))
                {
                    input.Angular("update-hidden");
                }
                return tag;
            });
        }
	}

	//Setup the conventions
	 public class ConfigTask_HtmlConventions
    {
        public static void Run()
        {
			//codedefined modifiers which mimic default asp.net mvc templating behaviour
			HtmlConventionsManager.LoadModule(new DataAnnotationModifiers(), new CommonEditorModifiers(), new SemanticModifiers(), new CommonDisplayModifiers());
					
			//load all conventions defined in the assembly
			HtmlConventionsManager.LoadModules(typeof (ConfigTask_HtmlConventions).Assembly);
			
			//codedefined builders for use with model annotations
			HtmlConventionsManager.LoadModule(new DataAnnotationBuilders(),new CommonHtmlBuilders());            
		}
	}
```
  While it looks a bit scary, it's codetty simple. We check that the model is a property and the parent or the property has [NgModel] and if that's the case, we modify the supplied HtmlTag: get the input element then add the ng-model attribute with the property name (made to be angularjs compatible). Then we handle some edge cases (these come out after you're using the app). In the end, we make sure that the hidden fields are updated, by adding the [_update-hidden_](http://www.sapiensworks.com/blog/post/2013/06/22/Binding-AngularJs-Model-to-Hidden-Fields.aspx) directive.

 In the view the code looks like this

  
```csharp
@Html.EditModel()
<!-- or -->
@Html.Edit(m=>m.MyProperty)
```
  Now, any view model decorated with [NgModel] is automatically angularjs ready.

 You can freely mix .Net Mvc's _Html.EditorFor()_ and _Html.Edit()_, any MvcPT helper which works with properties is compatible with the default html helpers.

 Let's see some Twitter Bootstrap integration. This automatically adds the classes _form-control_ and _form-group_ when rendering html text inputs.

  
```csharp
e.If(m => !m.IsRootModel)
	.Modify((tag, m) =>
	{
		var input = tag.FirstInputTag();
		if (input != null && input.IsHiddenInput()) return tag;
		var type = input.Attr("type");
		if (type != "checkbox" && input.TagName()!="textarea")
		{
			input.AddClass("form-control");
		}
		return tag;
	});

	e.If(m => 
	!m.IsRootModel && !m.Type.IsUserDefinedClass() && !m.PropertyOrParentHasAttribute<InlineAttribute>())
	.Modify((tag, m) =>

		new HtmlTag("div").AddClass("form-group").Append(tag)
	);
```
  These are example when you want to render input fields, forms but the same thing can be applied when you just want to display things. Here's a simple convention that says to render any property having [Text] as a _<code>_

  
```csharp
var d=conventions.Displays;
	
	d.If(m => m.PropertyOrParentHasAttribute<TextAttribute>())
                .Build(m =>
                {
                    return new HtmlTag("code").Text(m.ValueAsString);
                });
```
  Here we're building a display date time widget. Automatically, any DateTime property will be rendered with the widget.

  
```csharp
d.ForType<DateTime>()
                .Build(m =>
                {
                    var time = m.Value<DateTime>();
                    var tag = new HtmlTag("div")
                        .AddClass("last-update")
                        .Attr("title", time.ToLocalTime())
                        .Text(string.Format("Updated {0}", DateTime.UtcNow.Subtract(time).ToHuman()));
                    return tag;
                });
```
  In views

  
```csharp
@Html.DisplayModel()
	@Html.Display(m=>m.Property)
	
	//want to display a defined template like asp.net mvc does, but for ANY model
	@Html.DisplayTemplate(someModel)
	
	//Now you can do this
	@foreach(var item in Model.Items)
	{
		@Html.DisplayTemplate(item)
		//or if you want to use html conventions 
		@Html.Widget(item)
	}
```
  The cool thing here is that you don't need to create a partial (template) view for List<Bla> just to be able to use the templated html helpers.

 Whoa, that's a long post already and I've only scratched the surface of what you can do with html conventions. It's a codetty big and complex feature but it can give you a LOT of power and flexibility when building your views. To give you an idea, you have support for html conventions profiles that can be changed on every request and you can define your own conventions registries besides Editor, Displays or Label (but you need to implement your own html helpers for those).


