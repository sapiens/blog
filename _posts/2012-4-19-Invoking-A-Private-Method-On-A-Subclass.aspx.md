---
layout: post
title: Invoking A Private Method On A Subclass
category: .Net
---

I was just (re)watching Greg Young's [Event Sourcing Presentation](http://www.infoq.com/codesentations/Events-Are-Not-Just-for-Notifications) when the following code appeared on slides

  
```csharp
public abstract class Aggregate
 {
      private void ApplyChange(Event @event,bool isNew)
        {
            this.AsDynamic().Apply(@event);           
            if (isNew) _changes.Add(d);
        }
}
```
  Greg tells the audience that this method does some black magic(.Net only) to basically call an Apply method defined in a subtype, like this

  
```csharp
public class SubType: Aggregate
{
     private Apply(DeactivatedItemEvent @event)
	 {
	    _deactivated=true;
	 }
}

```
  Of course the scenario was that there are have many overloads of Apply, each for a different event. With this trick, you just write the method and that's it. The compiler and the runtime will work for you to select the correct overload.  
 So I was curious about the magic that made the call possible. You might think that a simple

  
```csharp
private dynamic AsDynamic() { return this; }

```
  is enough. But it isn't. In fact, it will throw an exception that the member is inaccessible. So what can we do? If you're browsing to Greg's CQRS application example you'll see some complicated implementation of DynamicObject which does 2 things: uses reflection (member.Invoke) to call the private method and does nothing if the method is missing. Now, the performance is good enough (around 1 second for 100 000 calls) but surely we can do better?

 After some messing around, I've come up with 3 ways to do it, besides reflection.

  
  2. The simplest way is to just make the method public , but that's kinda defeats the purpose of encapsulation. Not really an option. 
  4. Another way is to define an abstract method that will be implemented by the sub type  
```csharp

public class Aggregate
{
  private void ApplyChange(Event d,bool isNew)
        {
            ApplyX(d);
           
            if (isNew) _changes.Add(d);
        }

        protected abstract void ApplyX(Event d); 
}

public class SubType: Aggregate
{
     protected override void ApplyX(Event d)
        {
            Apply((dynamic)d);
        }
}

```
  Casting the argument to dynamic does the trick here and this is the fastest implementation (around 185 ms for 100 000 method calls). But you have to write that override for every subtype. Not a high price to pay to get the best performance, IMO.
     
       
  6. Write a dynamic method which will invoke a private method, directly without reflection  
```csharp

public class Aggregate
{
         protected void ApplyX(Event d)
        {
            var evt = d.GetType();
            lock (_locker)
            {
                if (!_invokers.ContainsKey(evt))
                {
                    DynamicMethod met = new DynamicMethod("invoker", typeof(void), new[] { typeof(EventEntityBase), typeof(Event) }, typeof(EventEntityBase).Module, true);
                    var il = met.GetILGenerator();
                    il.Emit(OpCodes.Ldarg_0);//subtype           
                    il.Emit(OpCodes.Ldarg_1);//event
                    il.Emit(OpCodes.Call, GetType().GetMethod("Apply", BindingFlags.Instance | BindingFlags.NonPublic, null, new[] { d.GetType() }, null));//call apply
                    il.Emit(OpCodes.Ret);
                    var del = (Invoker)met.CreateDelegate(typeof(Invoker));
                    _invokers[evt] = del;
                }
            }
            _invokers[evt](this, d);
        }


        static object _locker = new object();
        static Dictionary<Type, Invoker> _invokers = new Dictionary<Type, Invoker>();

        public delegate void Invoker(EventEntityBase bs, Event ev); 
}

```
  In a nutshell, a static method is created via Reflection.Emit, then used via a delegate, nothing fancy here. The delegate is cached for further reuse. This approach doesn't require anything implemented by the subtype so everything is contained by the base class. However its performance was a bit of surprise as it's slower than using dynamic directly (around 260ms for 100 000 method calls).   There you have it. You can choose the reflection way, which is the slowest or to use the dynamic feature of C# (almost 10x faster) but requires the subtype support or to emit a dynamic method ('only' 5x times faster). I think I'm going to use the approach with the dynamic keyword. 


