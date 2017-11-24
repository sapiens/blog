---
layout: post
title: When To Use The Static Keyword In .Net
category: .Net
---

f you come from a procedural language background or mindset, you'll find a very useful friend in the static keyword. Since C# and Vb.Net are OOP languages, having static methods and static fields in a utility class is the only way to mimic global functions. However, if you want to do good OOP and you aren't very experienced,  misusing '**static**' can lead to a headful (that's a pun) of problems.  
  
Take this code (actual code from a StackOverflow question)

  
```csharp
public static class dbConnection
{
    public static SqlConnection cn = new SqlConnection();

    public static void OpenConnection()
    {
        try
        {
            cn.ConnectionString = ConfigurationManager.ConnectionStrings["cnWebTwDrill"].ToString();

            if (cn.State == ConnectionState.Closed)
                cn.Open();
        }
        catch (Exception)
        {
            throw;             
        }
    }
}
```
  This is a good example of bad usage of _**static**_, especially since this code is from a web application (read: multi threaded application)! Being static, means that the same connection will be available as long as the application runs and that it will be shared and modified by every thread accessing it, which is a major concern in a high concurrency application. Lacking any thread safety mechanism means that threads will read and modify each other data in a non determinant way. So this is a case where you SHOULDN'T use _**static**_.  
  
In fact every time you're tempted to make something static you should think it thoroughly at least 2 times. The only case where you don't need to think about it is when defining extension methods which require static methods in a static class. But anything else is up for debate.   
  
That being said, when it's appropriate to use _**static**_? First candidates are methods which don't require state at all. They process their arguments as input and return an output. The best example is a utility method, but you should consider if that utility doesn't fit better as a behavior of an object. In almost all cases, a 'real' utility method can be used unmodified  as an extension method.  
  
Other important thing to be aware of is that anything static should take into account thread synchronization. A static field or property can be read or modified by any thread at random times and this should be handled properly and before you rush using locks everywhere, ask yourself again: 'Do I need this to be static?'. If so, make sure you deal with the synchronization issue.  
  
Another aspect is testing. By their nature, static classes and properties are against testing, since they maintain their state as long as the containing application runs. An utility method is easy testable, a class with a static field just messes up things. If you need it to be testable, avoid making it static.  
  
What about factory methods? A static factory method makes sense. The best usage is when you need to return an abstraction, a good example is an abstract class with a factory method which returns an instance of a derived class. But you can also use a static factory method to control the instantiation of a type, either to create an instance from another type ( example: the Int32.Parse method) or to validate a constructor argument gracefully without throwing exceptions (example: the Int32.TryParse method).  
  
As you can see, _**static**_ has pretty limited good usage and it's rather a trap if you're not experienced enough. So when you need a specific functionality, try first to see if it can fit an object, only afterwards decide if it helps more to be static. As a general principle avoid making static any code which needs disposing, which may take long to complete or which works with unmanaged resources.  
  
In a nutshell, if  
- functionality is utilitarian and it doesn't need state, or  
- factory method, or  
- needs to be accessed by multiple independent threads (like request processing in an asp.net application)  
then you have a good candidate for a static method or a class with at least a static field.  
  



