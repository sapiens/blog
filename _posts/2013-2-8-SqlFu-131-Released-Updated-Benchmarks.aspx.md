---
layout: post
title: SqlFu 1.3.1 Released, Updated Benchmarks
category: SqlFu
---

Version 1.3.1 of SqlFu brings  helpers to make some tasks easier. For writing moderately complex queries I codefer Sql (I think it's easier than messing with Linq) but for very simple queries or commands I wanted to remain in C# land. Doing inserts or updates is boring with sql and while there was an Update method in SqlFu which easied the pain, it wasn't enough because it couldn't handle trivial things like:

  
```csharp
update Table set column=column+1
```
  So I've included a very easy to use Update Table fluent builder

  
```csharp
db.Update<Table>()
.Set(t=>t.Column, t=> t.Column+1)
.Set(t=>t.OtherColumn, value)
.Where(t=>t.Id==someid)
.Execute()
```
  Simple enough and it should handle 99% of usages of single table updating.

 Other helpers added

  
  * retrieve one row using strongly typed criteria  
      
```csharp
db.Get<Article>(a=>a.Id==12)
```
   
  * get column value  
      
```csharp
db.GetColumnValue<Article,string>(a=>a.Title,a.Id==12)
```
   
  * counting rows from table  
      
```csharp
db.Count<Article>(a=>a.PostedAt.Year==2013)
```
   
  * checking if a table has rows  
      
```csharp
db.HasAnyRows<Article>(a=>a.Id==entityId);
```
   
  * delete from table  
      
```csharp
db.DeleteFrom<Article>(a=>a.Id=entityId);
```
   
  * some table management utils  
      
```csharp
db.Drop<Article>();
if (!db.TableExists<Article>()) { //do create table }
```
    Nothing revolutionary but small things to increase productivity.

 
## Benchmarks

 I thought it is a good time to  update the old benchmarks (published in May 2012) comparing SqlFu to other micro Orms. I've dropped FluentData and revised the queries as things were not that equal (some dapper queries included an order by, Massive query wasn't iterated) and I've updated all the other micro orm to their latest nuget versions.

  
```csharp
Executing scenario: FetchSingleEntity
-----------------------------------
Massive doesn't support the action. not explicit type support
Dapper query entity - 500 iterations executed in 65,7408 ms
InsightDatabase - 500 iterations executed in 66,2373 ms
OrmLite - 500 iterations executed in 66,4864 ms
SqlFu FirstOrDefault - 500 iterations executed in 67,734 ms
SqlFu Get - 500 iterations executed in 71,1035 ms
PetaPoco entity - 500 iterations executed in 72,9989 ms
Dapper get entity - 500 iterations executed in 97,0822 ms

Executing scenario: FetchSingleDynamicEntity
-----------------------------------
Dapper query entitty dynamic - 500 iterations executed in 66,4755 ms
InsightDatabase - 500 iterations executed in 67,662 ms
SqlFu dynamic - 500 iterations executed in 69,6545 ms
PetaPoco dynamic - 500 iterations executed in 71,9594 ms
OrmLite - 500 iterations executed in 72,5374 ms
Massive - 500 iterations executed in 107,5593 ms

Executing scenario: QueryTop10
-----------------------------------
Massive doesn't support the action. not explicit type support
InsightDatabase - 500 iterations executed in 90,5127 ms
Dapper  - 500 iterations executed in 91,2931 ms
PetaPoco - 500 iterations executed in 94,7761 ms
SqlFu - 500 iterations executed in 96,9167 ms
OrmLite - 500 iterations executed in 111,007 ms

Executing scenario: QueryTop10Dynamic
-----------------------------------
OrmLite - 500 iterations executed in 92,5813 ms
Dapper  - 500 iterations executed in 100,7141 ms
InsightDatabase - 500 iterations executed in 106,0291 ms
PetaPoco dynamic - 500 iterations executed in 114,4857 ms
SqlFu - 500 iterations executed in 127,215 ms
Massive - 500 iterations executed in 244,1929 ms

Executing scenario: PagedQuery_Skip0_Take10
-----------------------------------
Dapper  doesn't support the action. No implicit pagination support
InsightDatabase doesn't support the action. No implicit pagination support
PetaPoco - 500 iterations executed in 153,303 ms
SqlFu - 500 iterations executed in 173,2594 ms
OrmLite - 500 iterations executed in 191,4013 ms
Massive - 500 iterations executed in 247,4856 ms

Executing scenario: ExecuteScalar
-----------------------------------
Dapper scalar int - 500 iterations executed in 59,7867 ms
InsightDatabase - 500 iterations executed in 61,0439 ms
SqlFu scalar int - 500 iterations executed in 61,789 ms
PetaPoco int - 500 iterations executed in 63,7753 ms
OrmLite - 500 iterations executed in 68,7653 ms
SqlFu get scalar excodession based - 500 iterations executed in 71,3949 ms
Massive - 500 iterations executed in 98,7458 ms

Executing scenario: MultiPocoMapping
-----------------------------------
Massive doesn't support the action. Specified method is not supported.
OrmLite doesn't support the action. Suports only its own specific source format
Dapper  - 500 iterations executed in 67,3771 ms
InsightDatabase - 500 iterations executed in 67,3924 ms
SqlFu - 500 iterations executed in 73,1912 ms
PetaPoco - 500 iterations executed in 74,4355 ms

Executing scenario: Inserts
-----------------------------------
massive doesn't support the action. Couldn't figure how to insert pocos with auto increment id
InsightDatabase doesn't support the action. Specified method is not supported.
SqlFu - 500 iterations executed in 103,4232 ms
PetaPoco - 500 iterations executed in 116,122 ms
Dapper - 500 iterations executed in 116,6094 ms
OrmLite - 500 iterations executed in 135,619 ms

Executing scenario: Updates
-----------------------------------
InsightDatabase doesn't support the action. Specified method is not supported.
OrmLite - 500 iterations executed in 75,0417 ms
SqlFu - 500 iterations executed in 81,5845 ms
PetaPoco - 500 iterations executed in 84,1997 ms
Dapper  - 500 iterations executed in 129,9522 ms
massive - 500 iterations executed in 183,4292 ms
```
  It seems that performance is not that much of a concern when using a micro Orm. A few of miliseconds difference when dealing with 500 queries is negligible. Pretty much any micro Orm is fast these days so what will make you codefer one over the other is their feature set (did you know SqlFu has integrated support for [database migrations](https://github.com/sapiens/SqlFu/wiki) ?) and how comfortable you feel using them. And even if all seem to have similar core features, they do have different (not so obvious) strengths.


