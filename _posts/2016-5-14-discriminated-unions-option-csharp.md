---
layout: post
title: Using Discriminated Unions In C# 
category: .net
---

I've reached the point where I needed [discriminated unions](https://fsharpforfunandprofit.com/posts/discriminated-unions/) but in C#, which doesn't have support for it (yet). So, I've created a [couple of classes](https://github.com/sapiens/cavemantools/blob/master/src/CavemanTools/AnyOf.cs) (if you want a nuget they're part of my CavemanTools library) that simulate a basic discriminated union with 2 or 3 options.

```csharp

var fruits=new AnyOf<Apple,Orange>(new Apple());

fruits.When<Apple>(a=> {} );
fruits.When<Orange>(o=> {} );

//compile time type
var apple=fruits.As<Apple>();

//dynamic type

var apple=fruits.Value;

if (fruits.Is<Orange>()) {} 

``` 

Even if C# 7 will offer pattern matching, I still find useful to have a type that specifies and enforces only a couple of types; a dynamic field is just too generic.

And since option types won't be available in C# 7 , I've also included an [Optional](https://github.com/sapiens/cavemantools/blob/master/src/CavemanTools/Optional.cs) struct for those moments when you really don't want to deal with nulls .

```csharp
private Optional<User> GetUser(){ }

var user=GetUser();

if (user.IsEmpty) {} 

var other=user.ValueOr(new User());

```

P.S: Inb4 "Why don't you just use F# already?". Because I know C# better, my F# knowledge is limited at this point and I need these features NOW. Besides, I only want a couple of features, not the whole language.   