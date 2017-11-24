---
layout: post
title: The DI (IoC) Container Explained
category: Best Practices
---

While commenting on a stackoverflow question, Mark Seemann ([Autofixture](https://github.com/AutoFixture)) told me that in his experience a lot of people (even experienced developers) are not comfortable with using a DI Container, even if they understand the benefits. They seem to fear its "magic" .  This post is about dispelling the magic.

 I assume you already know what dependency injection (DI) is and you're already doing it manually. Something like

  
```csharp
var service= new Service(new Repository(new DBContext()), new OtherService( new OtherRepository()))
```
  Imagine doing that everytime you need an instance of that service or one of its dependencies. I know, we can create a Factory class that will take care of that. I got news for you: it's already created and it's called a DI Container (DIC). Yeah, a DIC is basically a factory for your objects. And it's not magical at all because it uses the language/framework reflection capabilities to know what are the dependencies of an object and also needs configuration in order to do something. And when magic needs configuration to work, it isn't magic.

 Let's take an example

  
```csharp
public interface IRepository { }
 
 public class Repository:IRepository {} 
 
 public class MyService{}
 
public class OtherService
{
    public OtherService(MyService svc, IRepository repo) {}

}
```
  We need an instance of OtherService so you ask the container for it. Some containers (like autofac) require explicit type registration for EVERY type that will be used by the container.If you ask it for an object will return null or throw an exception (not very magical). Other containers will try to create an instance using a public parameterless constructor (if available) and this is all the magic they do (calling_ Activator.CreateInstance()_ in .net).

 So first you need to tell the container that you want a certain type to be used and how it should be used. Basically, you say to it: "Know about this type and use it in the following scenarios: when a concrete instance is requested or when an interface implemented by this type is requested. Oh, and re-use the same instance every time for type A i.e make it a singleton".

 Let's say do this with Autofac

  
```csharp
var cb=new ContainerBuilder();
 
 cb.RegisterType<Repository>().AsImplementedInterfaces()
 
 //alternative more explicit 
 cb.RegisterType<Repository>().As<IRepository>().InstancePerHttpRequest();
 
 cb.RegisterType<MyService>();
 cb.RegisterType<OtherService>();
```
  Now, you ask for an object of **OtherService**. The Container sees the **MyService** dependency and uses the configuration to create an instance. Then it sees the **IRepository** dependency and it knows that for that interface it should use an instance of the **Repository**. If the Repository has itself a dependency, it will use the configuration to create an instance of that type. Once again, nothing mystical or magical here. Just a factory object using configuration and reflection to instantiate types.

 The point of the container is that it automates work for you. Instead of manually coding a factory, you configure an existing one. However in a real app, you won't be registering each type manually. Even here we can automate things :) using conventions. We decide on naming conventions and then we register types matching a convention in a certain way.

 For example, I decide that all my service will contain the word _Service_ in their names and all instances of a service should be request scoped (one instance per http request). However all my in memory repositories, should start with _InMemory_ and they should be singletons.

  
```csharp
cb.RegisterAssemblyTypes(this.Assembly).Where(t=>t.Name.Contains("Service")).AsSelf().AsImplementedInterfaces().InstancePerHttpRequest();
 cb.RegisterAssemblyTypes(this.Assembly).Where(t=>t.Name.StartsWith("InMemory")).AsSelf().AsImplementedInterfaces().SingleInstance();
```
  Now, everytime I add a new service or a new in memory repository, it gets automatically picked up by the container. Magic! Or not. The container scans for assemblies then asks them for the public types then filters them by a condition (implementing our convention) and registers the resulting types to be used as self and as concrete types for the implemented interfaces.

 A container also does object lifetime management so you don't have to worry about it. You can tell it to serve a new instance every time, an instance per scope (the http request is a tagged scope) or a singleton. Btw, this is the preferred way to implement singletons. At least Autofac cares about disposable types, so when the scope ends, Dispose() is automatically invoked.

 I hope you're not afraid of DI Containers any more. They are simply factories which instantiate classes and do the boring work of getting every dependency right and they need configuration in order to work properly. You know that objects should be loosely coupled and when you have many of them with abstractions and different concrete types for the same abstraction, the DI Container is an invaluable tool that simplifies your work. And once you've used one properly, it will be very hard not to use it from then on.

 P.S: I'm using a DI Container by default for every non-trivial (prototype,tests) app. It takes me a whole 30 seconds to set up one and then I just add configuration conventions as I need them. And I simply don't care about the numbers of classes or abstractions. I do care about the number of dependencies a constructor has but that's an object design issue, not a technical one.


