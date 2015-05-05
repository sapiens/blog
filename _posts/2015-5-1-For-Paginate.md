---
layout: post
title: aaaa

category: Other Stuff

---
The good news about writing a trivial app is that you don't need many things. Only some basic coding skills and a 'get it done' attitude. Design patterns, SOLID code etc are not required. It's code and forget. Easy and practical.

 However, when dealing even with moderately complex apps things are different: there is advanced functionality required and changes are the norm. Suddenly, the way the app is designed and how the code is written matters. At that point, we care a lot about maintainability, which implies decoupled code. And decoupled code means that most objects will take one or more abstract dependencies.

 So we have objects which require dependencies. That's not a problem as it's easy to provide those dependencies manually. Unless those dependencies themselves require dependencies and so on. At that point, a manual solution becomes cumbersome. A better, automated solution is needed and here is where the DI Container comes into play.

 There is a saying among developers: "When learning about DI Container you don't understand why you really need it, but after you start using it properly, you wonder how you managed without one".

 The most obvious benefit of a DI Container is the automated wiring of all dependencies. 1 or 100 it doesn't matter, the Container knows how to provide them. It's almost magical. But the benefits don't stop here.

 Fact is, a Container is a factory which creates objects on demand. But it doesn't just creates them, it can also manage their lifetime which is an important feature. You might be aware that the Singleton pattern can be viewed as an anti pattern, mainly because of the static property which messes up testing and multi-threading scenarios. Oh, and breaking the Single Responsibility Principle, because the object gets the additional task to handle its own lifetime.

 Using a Container, you just tell it to treat an object as a singleton so the same instance will be used every time. The object itself doesn't know that it'll be used as a singleton so there's no need for private constructors or static properties. The object is relieved of the task to manage its own lifetime. You get less code and increased testability.

 But that's not all, as you can set the Container to return the same instance for a defined scope, for example a request in a web app. Any time the object is needed in that scope, you know there will be the same instance. And speaking of scope, most Containers knows how to handle IDisposable, so when the scope ends any disposable object will be disposed as well.

 Containers are also useful when you have interfaces (abstractions) defined and you need instances of concrete but unkown types. This is the a very common usage in business applications which, for example, define abstractions for business rules then need the actual implementations. With a Container you just register all types implementing the interface (every Container has a method to scan and register types matching a criteria). Then the business rules engine asks the Container for instances of types implementing a certain interface (or just takes a dependency on a List<IBusinessRule>) and uses them. The benefit is that you can add lots of business rules implementations (new types) and they will be automatically used without needing to change or to add any other code. And the important detail is that at least some of the concrete objects have one or more dependencies.

 When you have a complex app, you have lots of code, especially lots of objects. A Container makes it easier to achieve low coupling and lets you focus on proper object design as you don't have to deal with object instantiation or dependency lifetime. Of course, you will instantiate manually a lot of objects, but none of them will be a dependency.

 Now, the most common antipattern when using a Container is something like this

  

```csharp
public class MyClass
{
   public MyClass()
   {
      var dep1=Container.Resolve<Dependency>();
   }
}

```
  This code invalidates the use of a Container because:

  
  2. **MyClass** is tightly coupled to the container 
  4. None of its dependencies are actualy injected  The proper code should look like this

  

```csharp
public class MyClass
{
   public MyClass(IDependency dep)
   {
     //assign dep to field
   }}

```
  **MyClass** doesn't know that a Container is used, it knows only about an abstract dependency. It's the usually the job of a framework to call the Container for an instance of **MyClass**. In your code you should try to refer to the Container as little as possible so that very little code should actually depend on the Container itself.

 Good code knows only about its immediate dependencies and those should be abstractions. When an object requires an outside object which is not injected as a dependency, you have a tight coupling problem. So use only the injected dependencies and let the Container do the wiring.

 P.S: Some Containers feature Interceptions which are used to implement Aspect Oriented Programming (AOP). In my opinion, this is an optional feature that isn't directly related to the main job a DI Container does. An AOP dedicated framework is a better choice.

