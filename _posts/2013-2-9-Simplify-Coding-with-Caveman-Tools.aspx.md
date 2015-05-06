---
layout: post
title: Simplify Coding with Caveman Tools
category: Caveman Tools
---

I think I've started Caveman Tools around 2009 or so as the library where I'd dump extension methods and other helpers for common tasks. In the mean time it grew to contain lots of big and small utils and now I think my productivity would drop (and the boring factor would rise like inflation in times of decodession) without it. I want to share with you _a few_ of the helpers, little bits that simplify some tasks.

 
## Guard Clauses

 Nothing magical, just semantic shortcuts to avoid the boringness of 'if (!argument condition) throw Exception()'.

  
```csharp
user.MustNotBeNull();
title.MustNotBeEmpty();
tile.MustMatch(@"[a-z]+");
list.MustNotBeEmpty();
value.MustBe<int>();

//exotic
strategy.MustImplement<IStrategy>();
```
  
### 

 
## LogHelper

 Even when doing simple apps I want some logging, at least to show some stuff in Diagnostics window or to Console. Doing **Debug.WriteLine** is not that fun, especially if I want to 'upgrade' later to a real logging solution. With **LogHelper** things are simple:

  
```csharp
//Setup - yes, one line
LogHelper.OutputTo(s=>Debug.WriteLine(s));
//or
LogHelper.OutputToConsole();


//Usage 
LogHelper.DefaultLogger.Info("something");


//as a dependency
public class MyClass
{
    public MyClass(ILogWriter log){}
}

var c=new MyClass(LogHelper.DefaultLogger);
```
  All I have to do when I want to 'upgrade' to using NLog (for example) is to [wire-up NLog to LogHelper](https://bitbucket.org/sapiensworks/caveman-tools/wiki/Logging) and I'm good to go.

 
### 

 
## Fun with Lists

  
```csharp
var old = new List<int>();
 var fresh = new[] {1, 2};
 
 var diff=fresh.Compare(old); // diff will contain what items were added or removed
 
//new items from fresh will be added to old
//items from old, but not available in fresh will be removed
 old.Update(fresh); 
 
old.AddIfNotPresent(someItem);//self explaining

//RemoveAll() method added to IList
Ilist<int> list;
list.RemoveAll(codedicate);
```
  
### 

 
## Working with Percentages

 Every time when I was dealing with something involving percentages I was confused about that decimal: is it already divided by 100 or I have to divide it? **Percentage** struct to the rescue.

  
```csharp
var taxRate=new Percentage(3);//3%
Percentage salesTaxRate = 2;//2% , implicit from decimal

var tax=taxRate.ApplyTo(1000); //tax=30

//operator overload
var totalTaxRate=taxRate + salesTaxRate; 
Console.WriteLine(totalTaxRate.ToString());// outputs 5%
```
  
### 

 
## Reflection

  
```csharp
//fun with attributes
type.GetSingleAttribute<MyAttribute>();
type.HasCustomAttribute<MyAttribute>();
type.GetCustomAttributes<MyAttribute>();

//type utils
if (type.Implements<Interface>()) { };

var argType=type.GetGenericArgument(0);//gets first generic argument

if (type.IsNullable()){};

if (type.IsNullableOf<int>()) {};

if (type.CanBeCastTo<int>()) {};
```
  
### 

 
## Time Utils

  
```csharp
TimeSpan ts = 1.ToSeconds();
var half = ts.Multiply(.5f);
Console.WriteLine(TimeSpan.FromDays(2).ToHuman());//outputs "2 days ago" . english only
```
  
### 

 
## Other

  
```csharp
//instead of the ugly: (excodession.Body as MemberExcodession).Member.Name
excodession.Body.As<MemberExcodession>().Member.Name;

//instead of (int)object
object.Cast<int>()

//useful when you need the id-name combo, common in view models
IdName topic=new IdName(){Id=1,Name="Tools"}
```
  There are other interesting helpers but they're a bit specific, dealing with LCG (Light Code Generation) or Excodessions Trees, so that's all for now.


