---
layout: post
title: The Repository Pattern Explained
category: Repository
---

**Update**: This is a much [better, less technical explanation](http://www.sapiensworks.com/blog/post/2014/06/02/The-Repository-Pattern-For-Dummies.aspx).

 Made popular by Domain Driven Design, the Repository pattern is one of my favourite design patterns, which I use almost everywhere. What can I say? I like clear separation between the database access and the rest of the application and this is the purpose of the Repository pattern.  
  
Separation of concerns is something you want in every real-life application and one of the most often met culprits when dealing with bad code is the happy mingleling of the database access  all over the place. Of course, this is also encouraged by the database centric approach which still dominates the developers mindset and you need stronger discipline in order to keep the database access isolated.  
  
In DDD, you start with the domain while the database access , in fact the persistence details, is ironed out at a later time. This means you are free to ignore anything persistence related, but  that doesn't mean that there is no persistence. However the persistence is viewed as a collection of objects where domain object are sent to die... sorry to be persisted and to be retrieved from. Note that the important thing is to consider the repository a collection and not to actually implement a collection (for example .net has an ICollection interface, this has nothing to do with a repository). This approach isn't something specific only to DDD, it can be used in any application as it is a design pattern after all.  
  
In a nutshell, the Repository pattern means abstracting the persistence layer, masking it as a collection. This way the application doesn't care about databases and other persistence details, it only deals with the abstraction (which usually is coded as an interface). The Repository also acts as a facade, simplifying the use of the persistence layer. The application just calls the methods of a repository in order to store or retrieve objects.  
  
Even if you don't do DDD, the objects you send or get from the repositories are the business/domain objects and **NOT** database related objects. This means that the repository implementation has knowledge of the business objects and knows how to recreate them.  
  
Using the Repository pattern doesn't mean there is only one repository in an application. There can be as many as needed, usually a repository deals with a specific context. For example, an application can have an OrdersRepository, an UsersRepository and an AdminRepository. It is good practice to code against an abstraction, so that's why you'll find many examples dealing with defining a repository interface and not a class. Working against an interface makes it easy to have multiple implementations which helps a lot with testing or changing implementation details.  
  
As an example, let's see how we can apply the pattern for a blog engine. When dealing with posts I can use this 

  
```csharp
public interface IPostsRepository
    {
        void Save(Post mypost);
        Post Get(int id);
        PaginatedResult<Post> List(int skip,int pageSize);
        PaginatedResult<Post> SearchByTitle(string title,int skip,int pageSize);
    }
```
  Post is a domain type, it isn't related to a possible Post class that can be defined in the persistence layer if you're using an OR\M. Remember that the Repository talks with the application only using the types that the application knows about. Any class defined for OR\M usage is hidden in the persistence layer as it's simply an implementation detail.  
      
    We see methods for saving a post or for retrieving a post. The Save method is smart enough to detect if it's a new post or a modified post. The PaginatedResult<Post>, is a C# generic type, meaning that the List and SearchByTitle methods return a list of Posts with pagination details.   
      
    When testing I can mock or stub this interface, without needing to actually implement a real database access. The application will work only with the interface (which will be injected where is needed), without caring about how things will be actually persisted.  
      
    There is no rule or format on how to define a repository. You define it according to what you need from it. But most of the time you'll need to Save or to Get objects in different ways.  
      
 A word about using transactions with a repository. Since most of the time the Repository is used in a DDD context, I'll refer to this case.  When you need to execute different operations as a unit, there's where the Unit of Work pattern comes into play. And there are discussion if the UoW is part of the Repository of viceversa. However, in DDD ,the repository should save only Aggregate Roots which themselves are a consistent unit. This means you don't need explicit transaction handling, because saving an aggregate root implicitly means that everything belonging to it is persisted as a transaction.


