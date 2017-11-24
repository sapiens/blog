---
layout: post
title: SqlFu 2.0 Released With Updated Benchmarks
category: SqlFu
---

Today marks the release of SqlFu 2.0 and some things are worth mentioning.

  
  * SqlFu now targets both .Net 4 and .Net 4.5 
  * Async support for .Net 4.5 
  * Breaking change: SqlFu now exposes functionality via extension methods of DbConnection (similar to dapper.net and others) 
  * Improved performance, especially in handling dynamic result, where it was much slower than other micro orms.  
## New benchmarks

 I added SimpleData to benchmarks, because it seems to be quite popular. The benchmark targets net 4.0.

  
```csharp
Executing scenario: FetchSingleEntity
-----------------------------------
SimpleData doesn't support the action. Specified method is not supported.
Massive doesn't support the action. not explicit type support
SqlFu FirstOrDefault - 500 iterations executed in 75,6658 ms
SqlFu Get - 500 iterations executed in 82,8085 ms
OrmLite - 500 iterations executed in 87,7121 ms
Dapper query entity - 500 iterations executed in 98,5202 ms
PetaPoco entity - 500 iterations executed in 102,525 ms
Dapper get entity - 500 iterations executed in 121,4952 ms
InsightDatabase - 500 iterations executed in 161,5671 ms

Executing scenario: FetchSingleDynamicEntity
-----------------------------------
SqlFu dynamic - 500 iterations executed in 79,0377 ms
OrmLite - 500 iterations executed in 81,8827 ms
InsightDatabase - 500 iterations executed in 83,8914 ms
PetaPoco dynamic - 500 iterations executed in 94,9423 ms
Dapper query entitty dynamic - 500 iterations executed in 96,2082 ms
Massive - 500 iterations executed in 129,1116 ms
SimpleData dynamic - 500 iterations executed in 133,3326 ms

Executing scenario: QueryTop10
-----------------------------------
SimpleData doesn't support the action. Specified method is not supported.
Massive doesn't support the action. not explicit type support
SqlFu - 500 iterations executed in 108,9925 ms
Dapper  - 500 iterations executed in 114,298 ms
PetaPoco - 500 iterations executed in 116,3961 ms
InsightDatabase - 500 iterations executed in 134,9926 ms
OrmLite - 500 iterations executed in 143,5561 ms

Executing scenario: QueryTop10Dynamic
-----------------------------------
OrmLite - 500 iterations executed in 114,2205 ms
SqlFu - 500 iterations executed in 117,3217 ms
Dapper  - 500 iterations executed in 126,3814 ms
InsightDatabase - 500 iterations executed in 141,544 ms
PetaPoco dynamic - 500 iterations executed in 143,2506 ms
Massive - 500 iterations executed in 327,3224 ms
SimpleData - 500 iterations executed in 405,4919 ms

Executing scenario: PagedQuery_Skip0_Take10
-----------------------------------
Dapper  doesn't support the action. No implicit pagination support
InsightDatabase doesn't support the action. No implicit pagination support
SqlFu - 500 iterations executed in 228,6695 ms
OrmLite - 500 iterations executed in 250,8005 ms
Massive - 500 iterations executed in 318,8973 ms
SimpleData - 500 iterations executed in 530,0301 ms
PetaPoco - 500 iterations executed in 1035,1179 ms

Executing scenario: ExecuteScalar
-----------------------------------
Dapper scalar - 500 iterations executed in 72,4849 ms
InsightDatabase - 500 iterations executed in 77,5271 ms
SqlFu scalar - 500 iterations executed in 82,1141 ms
SqlFu get scalar expression based - 500 iterations executed in 84,9293 ms
OrmLite - 500 iterations executed in 87,2419 ms
PetaPoco string - 500 iterations executed in 87,8697 ms
Massive - 500 iterations executed in 124,431 ms
SimpleData scalar - 500 iterations executed in 154,2036 ms

Executing scenario: MultiPocoMapping
-----------------------------------
SimpleData doesn't support the action. Specified method is not supported.
Massive doesn't support the action. Specified method is not supported.
OrmLite doesn't support the action. Suports only its own specific source format
SqlFu - 500 iterations executed in 88,1978 ms
Dapper  - 500 iterations executed in 88,6813 ms
PetaPoco - 500 iterations executed in 97,1947 ms
InsightDatabase - 500 iterations executed in 97,8031 ms

Executing scenario: Inserts
-----------------------------------
InsightDatabase doesn't support the action. Specified method is not supported.
massive doesn't support the action. Couldn't figure how to insert pocos with auto increment id
SqlFu - 500 iterations executed in 138,7476 ms
PetaPoco - 500 iterations executed in 153,3651 ms
Dapper - 500 iterations executed in 169,0831 ms
OrmLite - 500 iterations executed in 188,4134 ms
SimpleData - 500 iterations executed in 362,3814 ms

Executing scenario: Updates
-----------------------------------
InsightDatabase doesn't support the action. Specified method is not supported.
OrmLite - 500 iterations executed in 104,1065 ms
PetaPoco - 500 iterations executed in 107,2367 ms
SqlFu - 500 iterations executed in 108,4179 ms
Dapper  - 500 iterations executed in 188,0426 ms
massive - 500 iterations executed in 238,6045 ms
SimpleData - 500 iterations executed in 263,6972 ms
```
  
### **Interesting bits**

  
  * SqlFu dynamic result performance is lacking consistency. Sometimes it has the best performance, but other times is in the middle.  
  * SqlFu PagedQuery, Insert and QuerySingle are consistently 10-15% faster than the other micro orms.  For whatever reason, PetaPoco (5.0.1) PagedQuery is **very** slow. Maybe it's a bug?! 
  * SimpleData is cool, but... it was so weird to write the queries. I still haven't managed to get it how to return only some columns for one result. It doesn't seem to like the way I wrote the query and I don't have the time to learn it properly . It's cool but it's also magical, different and slow. Actually the slowest of the bunch. So you'd be using it for the magic part and not for performance. 
