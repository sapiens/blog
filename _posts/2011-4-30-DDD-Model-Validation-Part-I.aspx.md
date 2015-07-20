---
layout: post
title: DDD Model Validation (Part I)
category: .Net
---

In my [previous post](http://www.sapiensworks.com/blog/post/2011/04/16/Whats-Wrong-With-Model-Validation-In-AspNet-Mvc.aspx) I wrote about the shortcoming of the default validation of Asp.Net MVC. That doesn't mean though, that a Asp.Net MVC aplication can't use a different validation approach. Phil Haack and his team implemented this way as a helper to cut down some of the boring repetitive tasks. But when dealing with a complex application there are simply better ways to do it in order to mantain better code quality and in this post I'll show you one.

 Keep in mind that I'm using a simple example and this approach might seem overengineered but the purpose is to illustrate some concepts.

 Let's say we want to make a blog (yes, the standard demo nowadays) and of course we need to model the posts. Usually you will have a Post entity , but let's think a bit. What do I need to show on a post? Something like that:

 - PostId  
- Publishing Date  
- Title  
- Content  
- Tags  
- Author Name  
- Author Id  
- List of categories names and id  
- Number of comments  
- List of comments  
- Number of views

 I can create a Post entity that will contain all those properties, but when I'm creating or editing a post, what do I need? First some properties:

 - PostId (null if creating)  
- Title  
- Content  
- Tags  
- Categories Id  
- AuthorId  
- PublishingDate  
- Is Published

 and I also need to have validation and further processing behaviour. It's obvious that while we have most of the same properties, there are differences between what I need when creating/updating the post and what I need to display it.

 So here is the **first principle**: we have different models for either create/update (command) and select (query). Each model has common properties but that's about it. If we try to have only 1 model then you'll either have a weak model (simple structure) for write or more complex that it's needed  for read. It's cumbersome to force a single model for different purposes so the first step is to separate them. We'll have a rich model for create/update and a lightweight model (simple POCO) for displaying.

 Since we're dealing with the validation topic the focus is on the rich model, the PostEntity might look like this

 

  
```csharp
public class PostEntity
{
    public int Id {get;set;}

    [Required]
[StringLength(100)]
public string Title {get; private set;}

[Required]
public string Content {get; private set;}

public DateTime PublishedDate {get; set;}

public bool IsPublished {get;set;}

private PostEntity() {}

public PostEntity CreateFrom(CreatePostData data, IValidationDictionary errors)
{
   if (data.Validate<PostEntity>(errors)) 
{
   var post= new PostEntity(){Title=data.Title, Content=data.Content, IsPublished=data.IsPublished};
 return post;
} 
return null;
}
 
}
public class CreatePostData
{
public string Title {get;set;}
public string Content {get;set;}
public bool IsPublished {get;set;}
}
```
  

 I'm keeping it at a minimum so that it's easier to follow. The model looks codetty ordinary, but you can see that you annotated the entity instead of the viewmodel (as you would have done normally). The CreatePostData (the view model)  is simply a DTO which will be used to create the PostEntity. The entity knows how to validate itself with the same DataAnnotations (i'm using my [CavemanTools](http://cavemantools.codeplex.com/) validator helper for that).

 However you can see something unusual: the constructor is private and we have a static factory method to create the entity. Why is that? The purpose is to ensure that the enitity is always in a valid state and we also want to return error messages to the user. The factory method validates the data according to validation attributes and then it returns a new PostEntity if everything is ok or null if there are errors.

 And that's the **second principle**: an object should be always in a valid state or it shouldn't exist at all. This principle ensures that we don't have to ask ourselves if we validated the object nor we need a _IsValid_ property to check and that alone  helps us avoid potential bugs. It also means that you don't have to rely on an external service to validate. For example, the Post Entity can be used without change with asp.net web forms or a web service.

 But that's not all, in the next post we'll see how we can refactor the entity to give our model more flexibility.


