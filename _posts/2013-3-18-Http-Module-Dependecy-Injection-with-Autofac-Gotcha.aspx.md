---
layout: post
title: Http Module Dependecy Injection with Autofac Gotcha
category: Best Practices
---

Using a HttpModule with constructor dependency is not a problem with [HttpModuleMagic](http://nuget.org/packages/HttpModuleMagic/) . What is a problem is that the modules are treated as singletons and are shared by all requests. And when that dependency is a repository which uses a database, you have a more subtle problem. What can you do?

 First of all, the dependency must become a factory (method) so that every time you need an instance of the repository, you invoke the factory. The interesting fact is that the factory instance will be treated as a singleton as well. Using Autofac i've registered the repository this way.

  
```csharp
builder.RegisterType<UserRightsRepository>().As<IUserRightsRepository>();
```
  The http module has the constructor

  
```csharp
public UserRightsModule(Func<IUserRightsRepository> repository)
{
}
```
  

 Autofac knows how to supply the Func dependency without other configuration, however there is a problem. First time, everything works well, but refresh the page and you get a yellow screen. Something about _lifetime scope being disposed_. The gotcha is that Autofac container which supplies the dependency is valid ONLY for the first request. Then it gets disposed, but the module still has a reference to it: the Func<>.

 To solve this problem we need to do this

  
```csharp
builder.Register<Func<IUserRightsRepository>>(cc =>
                {
                    return () =>
                        AutofacDependencyResolver.Current.RequestLifetimeScope.Resolve<IUserRightsRepository>();
                });
```
  We configure Autofac to return this lambda which will always use the **current request scoped container** to create the repository. That is, for each request there is another container which will supply the dependency. Tricky stuff, read more about [Autofac lifetime scope here](http://nblumhardt.com/2011/01/an-autofac-lifetime-primer/) .


