---
layout: post
title: The Generic Repository Is An Anti-Pattern
category: Repository
---

It seems to me that when you search about repository pattern, you inevitable stumble upon the generic repository, touted as how a repository should be implemented. Too bad that's a fallacy. A repository is a concept to abstract the access to the persistence, that is not to depend on data access implementation details. There is no formula and no rules.  
  
A repository is also the contract used by the rest of the application to save/load objects.  Every application is different and has different needs of accessing the persistence. And yet , many advise you to use a generic repository like this

  
```csharp
public interface IRepository<T>
 {
    IEnumerable<T> GetAll();
    T GetByID(int id);   
    void Add(T entity);
    void Update(T entity);
    void Delete(T entity);
 }
```
  One of the touted benefits is the DRY (Don't Repeat Yourself) principle. It's good we don't apply the principle when talking, I mean it would be hard not to repeat the same sounds or words. Sadly, for some it seems that every application has the same persistence access needs, because that's the only scenario where a generic repository makes sense.   
   
 I'm aware that a Save , Get or Update method are common in a repository but it depends... . Am I dealing with a query repository or a command one? Is it a repository specialized for reading or for updating model? Because a generic repository can't handle those cases. In one repository you might need GetById and GetByTitle with pagination support. In other you might need only a simple Get method. Other repository might deal only with Save/Update a domain entity.  
   
 What I'm trying to say is that the design of a repository interface depends too much on what bounded context it applies for, so there isn't a generic form. If you find that things are repeating it might be a sign of poor architecture or that specific application is really suitable for something generic. But that's a specific case, not the rule.  
   
 Other offender in regard to generic repositories is the fact that lots of developers just use it to wrap the DAO (Database Access Object) or an underlying ORM (like EF or Nhibernate). Doing so they add only a useless abstraction, codetty much just making the code more complex with no benefits. A DAO makes it easy to work with a database, an ORM makes it easy to access a database as an OOP virtual storage and to eventually abstract the access to a specific database.   
   
 But the repository should abstract the whole persistence layer, hiding implementation details like database engine or what DAO or ORM the app is using but also providing a contract that makes sense from the application point of view. The repository serves the application needs, NOT the database needs.  
   
 A generic repository serves the dogma because very few applications have a need for a generic contract applicable everywhere and when used just to mask a DAO, it becomes an unnecessary abstraction. These two reasons make the Generic Repository pattern an anti pattern.

 **Update**: A generic repository fits almost naturally as a domain repository, but **only** as that. To use it anytime you want a repository and especially when dealing with query repos doesn't make sense.


