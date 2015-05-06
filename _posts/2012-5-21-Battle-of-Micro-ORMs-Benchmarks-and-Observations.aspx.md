---
layout: post
title: Battle of Micro-ORMs, Benchmarks and Observations
category: SqlFu
---

**Update**: See [this post ](http://www.sapiensworks.com/blog/post/2013/02/08/SqlFu-131-Released-Updated-Benchmarks.aspx)for newer benchmarks

 Since I did a bit of benchmarking for my [SqlFu micro-orm](http://www.sapiensworks.com/blog/post/2012/05/19/SqlFu-My-Versatile-Micro-Orm.aspx), I thought how about I'd do a 'proper'  job and benchmark more micro-orms and more usage scenarios. I've included all the micro-orms I found via Nuget or I knew about, but I didn't include any of the heavy ORMs as they're aimed at a different target .

 The participants:  
- SqlFu   
- Dapper.net with extensions  
- PetaPoco  
- Massive  
- FluentData  
- InsightDatabase  
- ServiceStack.OrmLite

 
## A few observations

 Because every orm has its quirks, I had to remove enum handling and avoid returning null when the result is a nullable value.

 **PetaPoco**  
 -  doesn't like converting from tinyint to Enum when executing scalar.   
 -  doesn't support mapping from string to Enum  
 -  doesn't suport mapping dbnull to nullable when executing scalar  
 -  handles up to 5 pocos and requires to put the column names in the same order as the properties of the result poco. A bit awkward but it works.

 **Dapper**  
- it doesn't support byte to enum when mapping result  
- the multi mapping kinda works or I don't know how to write the sql in order for dapper to properly map it. Anyway, this is an issue for anyone using it: hit or miss depending on the sql.

 **Massive**  
- it's strange to work with, perhaps because I am not used to the Active Record pattern. I consider it a bit unintuitive to use compared to the others.  
- it can't automatically map scalars to a type, you have to do the conversion yourself.  
- couldn't figure how to do a proper insert

 **FluentData**  
- quite intuitive, I like using it  
- doesn't automatically ignore properties not found in query result  
- doesn't support dbnull to nullable when executing scalar  
- doesn't support  byte to enum  
- doesn't handle Inserts properly - SqlServer complains about the generated sql.

 **ServiceStack OrmLite**  
- it does support a form of multi mapping but it is specific to OrmLite.  
- doesn't support Enum to byte (it's safe to assume it requires enums to be persisted as varchar)

 **InsightDatabase**  
- built mainly for stored procedures usage, annoying if you're using just sql  
- interesting features (like async queries) which are outside of this benchmark

 **SqlFu**  
- Handles Enums properly and it doesn't choke on nullables. It's also the most handsome of them all.

 
## Scenarios

 Each use case is run 500 times, after 10 times of warm up. When the micro-orm doesn't support a feature out of the box (i.e you'd have to do it manually), I put it as Not supported action. Only suported actions are timed. I've run all the benchmarks a couple of times in order to get relevant results.

 While there is no clear winner, especially since you have to consider **all the features** a package offers, you can clearly see a trend on who's in the top and who's lagging behind.  
  
Executing scenario: **FetchSingleEntity**  
-----------------------------------  
Massive doesn't support the action. not explicit type support  
Dapper entity - 500 iterations executed in 76,5396 ms  
SqlFu Get - 500 iterations executed in 77,6155 ms  
PetaPoco entity - 500 iterations executed in 78,8191 ms  
SqlFu FirstOrDefault - 500 iterations executed in 80,1971 ms  
OrmLite - 500 iterations executed in 80,2547 ms  
InsightDatabase - 500 iterations executed in 160,2398 ms  
FluentData - 500 iterations executed in 206,3038 ms  
  
Executing scenario: **FetchSingleDynamicEntity**  
-----------------------------------  
SqlFu dynamic - 500 iterations executed in 74,6951 ms  
Dapper dynamic - 500 iterations executed in 80,3824 ms  
PetaPoco dynamic - 500 iterations executed in 83,6375 ms  
Massive - 500 iterations executed in 130,8806 ms  
OrmLite - 500 iterations executed in 146,8522 ms  
InsightDatabase - 500 iterations executed in 176,2635 ms  
FluentData - 500 iterations executed in 280,377 ms  
  
Executing scenario: **QueryTop10**  
-----------------------------------  
Massive doesn't support the action. not explicit type support  
SqlFu - 500 iterations executed in 90,7809 ms  
PetaPoco - 500 iterations executed in 106,1067 ms  
Dapper  - 500 iterations executed in 115,7401 ms  
OrmLite - 500 iterations executed in 128,891 ms  
InsightDatabase - 500 iterations executed in 203,417 ms  
FluentData - 500 iterations executed in 445,0844 ms  
  
  
Executing scenario: **QueryTop10Dynamic**  
-----------------------------------  
Dapper  - 500 iterations executed in 99,1737 ms  
OrmLite - 500 iterations executed in 108,785 ms  
Massive - 500 iterations executed in 116,3599 ms  
SqlFu - 500 iterations executed in 116,4271 ms  
PetaPoco dynamic - 500 iterations executed in 125,1672 ms  
InsightDatabase - 500 iterations executed in 192,129 ms  
FluentData - 500 iterations executed in 198,8402 ms   
  
Executing scenario: **PagedQuery_Skip0_Take10**  
-----------------------------------  
Dapper  doesn't support the action. No implicit pagination support  
FluentData doesn't support the action. No implicit pagination support  
OrmLite doesn't support the action. No implicit pagination support  
InsightDatabase doesn't support the action. No implicit pagination support  
Massive - 500 iterations executed in 115,4747 ms  
SqlFu - 500 iterations executed in 199,1776 ms  
PetaPoco - 500 iterations executed in 292,4833 ms   
  
Executing scenario: **ExecuteScalar**  
-----------------------------------  
OrmLite - 500 iterations executed in 59,7584 ms  
InsightDatabase - 500 iterations executed in 64,2303 ms  
PetaPoco int - 500 iterations executed in 66,3844 ms  
SqlFu scalar int - 500 iterations executed in 77,7015 ms  
Dapper scalar int - 500 iterations executed in 83,0212 ms  
Massive - 500 iterations executed in 130,4026 ms  
FluentData - 500 iterations executed in 157,6623 ms  
  
Executing scenario: **MultiPocoMapping**  
-----------------------------------  
Massive doesn't support the action. Specified method is not supported.  
OrmLite doesn't support the action. Suports only its own specific source format  
InsightDatabase doesn't support the action. No implicit multi mapping support  
SqlFu - 500 iterations executed in 75,7716 ms  
Dapper  - 500 iterations executed in 88,3029 ms  
PetaPoco - 500 iterations executed in 99,3068 ms  
FluentData - 500 iterations executed in 236,9429 ms  
  
Executing scenario: **Inserts**  
-----------------------------------  
Massive doesn't support the action. Couldn't figure how to insert pocos with auto increment id  
FluentData doesn't support the action. Specified method is not supported.  
InsightDatabase doesn't support the action. Specified method is not supported.  
SqlFu - 500 iterations executed in 112,067 ms  
PetaPoco - 500 iterations executed in 126,0432 ms  
OrmLite - 500 iterations executed in 128,8205 ms  
Dapper - 500 iterations executed in 264,1543 ms   
  
Executing scenario: **Updates**  
-----------------------------------  
InsightDatabase doesn't support the action. Specified method is not supported.  
SqlFu - 500 iterations executed in 75,1604 ms  
PetaPoco - 500 iterations executed in 80,522 ms  
Dapper  - 500 iterations executed in 102,6581 ms  
OrmLite - 500 iterations executed in 195,2351 ms  
massive - 500 iterations executed in 202,5537 ms  
FluentData - 500 iterations executed in 249,3975 ms   
  
  
You can find the benchmark project on [GitHub](https://github.com/sapiens/SqlFu) (part of the SqlFu repository).


