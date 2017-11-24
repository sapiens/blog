---
layout: post
title: DRY Asp.Net Mvc Controllers
category: Caveman Tools
---

Since reading [Jimmy Boggard's post](http://lostechies.com/jimmybogard/2011/06/22/cleaning-up-posts-in-asp-net-mvc/), I've been trying to DRY things in the Asp.Net Mvc land. Truth is, in every POST action you have to verify if the model is valid. Is not much code but it's repetitive and you do it because you have to. It's not like you can continue the action with an invalid model.

 What if the controller would verify automatically that the model is valid and if it isn't , to return directly the view. So the controller action wouldn't be even called for an invalid model.

 That is instead of this

  
```csharp
public class HomeController:Controller
{

  [HttpPost] 
 public ActionResult CreatePost(Post model)
  {
    if (!ModelState.IsValid)
    { 
        model.Categories=GetAllCategories();  
        return View(model);
    }

    ProcessAndSaveModel(model);
    
    if (someError)
    {
       ModelState.AddModelError("key","something happened");
       model.Categories=GetAllCategories();
       return View(model);
    }
    return RedirectToAction("index");
  }

}
```
  we would have this

  
```csharp
[SmartController]
public class HomeController:Controller
{
  
 [HttpPost]
 public ActionResult CreatePost(Post model)
 {
   ProcessAndSaveModel(model);

   if (someError) ModelState.AddModelError("key","something happened");

  return RedirectToAction("index");
 }
 
}
```
  Instead of 9 lines of code, we have now only 3. We've reduced the action size by 2/3! All just by decorating the controller with the SmartControllerAttribute. Pretty good!

 In fairness, there is a bit more code involved but not as part of the controller. Some functionality was moved outside the controller but this just makes things more decoupled. The point is that in the first example we have 2 things repeating themselves:

  
  * checking if the model is valid and 
  * populating the model before returning the View result.  In the second example, nothing repeats itself. Both examples have identical functionality.

 Before diving in, let me tell that the SmartController is available as part of the newly released [CavemanTools.Mvc](https://bitbucket.org/sapiensworks/caveman-tools/wiki/CToolsMvc) (use Nuget to get it).

 Although you've seen the SmartController, the actual feature is in fact implemented by the SmartActionAttribute. As you might have guessed the difference between them is that one works at the controller level (applies to all actions) while the other works at the action level (only the decorated action gets the feature).

 How does it work? The Smart functionality is an action filter running before and after the action. It checks the modelstate validity and if it fails it uses a predefined policy, which by default is to return a view with the same name as the action. If before the action, the action won't be invoked at all. If the validation fails after the action, then the result returned by the action is ignored.

 So you don't have to do the checks yourself. The action will be invoked ONLY if validation succeeds and the result will be executed ONLY if the second validation succeeds.

 What about the case where a model needs to be populated? In many scenarios, the view model needs more than the info received from the request. In this example, our Post model needs the categories set. How does the magic works?

 When preparing the model for a failed validation, the SmartAction checks if there is a class which can populate the model. That is, it asks the DI Container if it has an ISetupModel<T>, in this case an ISetupModel<Post>, like this (a trivial example, an actualy setup usually involves more than 1 line)

  
```csharp
public class PostModelSetup:ISetupModel<Post>
{
  IRepository _repo;
   public PostModelSetup(IRepository repo) 
  {
    _repo=repo;
  }
 public void Setup(Post post)
 {
     post.Categories=_repo.GetAllCategories();
 }
}
```
  If it exists, the object is used to populate the categories automatically. This is the functionality removed from the initial controller. Now we can use it anywhere we need to setup the Post model. When a GET request comes for the CreatePost we use this object to create the Post model like this

  
```csharp
[SmartController]
public class HomeController:Controller
{
  /* .. Constructor with dependencies ... */
  
  
  public ActionResult CreatePost()
 {
  var model= new PostModelSetup(repository).Create();
  model.SelectedCategoryId= 2;
  return View(model);
 }


//This action is unchanged 
 [HttpPost]
 public ActionResult CreatePost(Post model)
 {
   ProcessAndSaveModel(model);

   if (someError) ModelState.AddModelError("key","something happened");

  return RedirectToAction("index");
 }
}
```
  We achieved DRY for the setup of the model also. Without these, you would have to repeat yourself in 3 places: once in the GET CreatePost and twice in the POST CreatePost.

 You can read about more [Smart features here](https://bitbucket.org/sapiensworks/caveman-tools/wiki/SlimControllers). Keep your controllers DRY!


