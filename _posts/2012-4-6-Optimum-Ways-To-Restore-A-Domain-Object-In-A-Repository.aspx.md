---
layout: post
title: Optimum Ways To Restore A Domain Object From Persistence
category: Best Practices
---

**Update** [Strategies To Restore Domain Entities ](http://www.sapiensworks.com/blog/post/2014/05/17/Strategies-To-Restore-Domain-Entities-.aspx)

 Let's say we have this object (the actual definition is not important)

  
```csharp
public class QueueItem
    {    
        public QueueItem(int blogId,string name,Type command,object data)
        {
            if (name == null) throw new ArgumentNullException("name");
            if (command == null) throw new ArgumentNullException("command");
            BlogId = blogId;
            CommandType = command;
            ParamValue = data;
            CommandName = name;
            AddedOn = DateTime.UtcNow;
        }

      
        public Guid Id { get; internal set; }
        public int BlogId { get; private set; }
        public string CommandName { get; set; }
        public Type CommandType { get; private set; }
        public object ParamValue { get; private set; }
        public DateTime AddedOn { get; private set; }
        public DateTime? ExecutedOn { get; private set; }
        public void ExecuteIn(ILifetimeScope ioc)
        {
            throw new NotImplementedException();
        }
    }
```
  This domain object will be handled by a repository

  
```csharp
public interface IRepository
{
     void Save(QueueItem item);
     QueueItem Get(Guid id);
}
```
  The repository persists the object, but also has to restore it. And here lies our problem. When implementing the repository, how can we restore the domain object, respecting the encapsulation and without  requiring the object to provide special behaviour for this task.

 It turns out that it depends.

 Now, when designing the domain object, the design _should not care_ about the persistence needs, it isn't the object's responsibility to know about storage, serializing and deserializing, that's the repository's concern. In this example, when a QueueItem is created, some values are supplied, but the **AddedOn** property is automatically set by the object. When restoring the object, the repository obviously can't use that constructor, so it needs a way to fill in all the values.

 There are several ways, which are most obvious but all have the same flaw: they require that the object change just for this purpose and that's not good.

  
  2. **Having another constructor** that has all the values as arguments.  
     This solution may work,  but:  
       
       * That constructor is available everywhere and you'd have to specify in comments that that constructor is to be used only for restoring the object. Pretty ugly. 
       * If the object has many fields then the constructor needs to accept LOTS of arguments, which is not a pretty sight.   
  4. **Making all the fields/properties setters public**.   
     This is probably the worst solution, as it breaks the encapsulation. **Don't do it! Ever!** 
  6. **Provide a factory method**.  
     This is actually the best solution from this group. The domain will keep its encapsulation and the static method won't interfere with the actual usage. Also it provides a very clear intent  
       
```csharp
public static QueueItem Restore(Guid id,int blogId /* etc*/) {}
```
  However is still has the same parameters number problem as the constructor solution, but at least it depends form object to object.
     
        The good news is there still are other better solutions:

  
  2. If you're using a document database (like RavenDb) or you're storing the objects in serialized form (I recommend JSON for that) just deserialize it back or let the db do it. But most of the time you won't be using a document database nor you'll be storing the objects in JSON. However it's a good solution, if the conditions are met. 
  4. Mapping. Using a mapper (I use Automapper) the domain object is created and its data its copied automatically from a provided source.  These solutions don't require to change anything in the domain object definition so the class can evolve according to the domain needs, ignoring anything persistence related (so the separation of concerns is maintained).

 You might have noticed that I didn't include the case when an ORM is used. That's because an ORM is an implementation detail of the repository and the **Domain Model is different from the Persistence Model** handled by the ORM (I don't consider relevant the cases where those happen to be identical because of a very simplistic domain or a misuse). And being different means you still have to transfer the values from the ORM entities to the domain object.

 But what if you can't use any of the good solutions above? No document db and no automapper (I find it hard to believe automapper can't be used almost anywhere but it may be the case). Then the next optimum solution is to use the factory method I've presented before. It's not the best but it's a good compromise. Of course, if the object is very complex perhaps a dedicated factory is a better choice.


