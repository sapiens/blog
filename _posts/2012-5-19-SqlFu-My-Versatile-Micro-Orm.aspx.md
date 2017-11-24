---
layout: post
title: SqlFu My Versatile Micro-Orm
category: SqlFu
---

I can already hear you: "What, another micro-orm, do we really need yet ANOTHER ONE?". Yes and believe me I've tried! I've searched for a micro-orm that will have those 2 features I really wanted. OK, at least one of them: multi poco mapping by convention. I wanted that for my view models. It's really tedious to write glue code or to mutate my models so they play nice. And I found it! FluentData was doing  multi poco mapping similar to what EF does: just name the columns [Property]_[Property] and the automapper will take care of it.

 Such bliss! Until I've benchmarked it... Then everything shattered. FluentData is SLOW (well compared to PetaPoco - my micro-orm of choice) and features some weird (IMO of course) design choices. Plus, I couldn't find a way how to do paged queries, but the slowness part was the deal killer. On the other hand it did allow you to use your very own mapper when you wanted - I liked that , but it was also the only way to be fast.

 What to do, what to do? I've thought well, since PetaPoco does suport a way to map multiples poco, I can add that functionality to it. And I did it, as a proof of concept but it was slow (reflection based). And my other pet peeve: I couldn't have both multi poco mapping (MPM) AND pagination in the same query. And the MPM required to put the column names in a certain order and it would support only up to 5 pocos (same as Dapper.net I think).

 Long story short, I've decided to develop my own micro-orm -SqlFu - which will be fast, it will support MPM automatically, no other convention or setup required but the columns names to contain the properties separated by '_' ,  pagination no matter if it's a simple object mapping or a MPM. Basically, I wanted a library that will do automatically the basic things and to be versatile when needed. It really puzzles me why there aren't many options like it already. I mean, in any web scenario you will have pagination and composed view models, why should I write that boring paging syntax (no fun on SqlServer) or poco mapping?! Yes, Automapper can be a solution but it is yet an abstraction layer: get poco from db ,then map it to another poco which will contain pretty much the same properties albeit in a an hierarchy. I think this multipoco mapping feature should be default in any self-respecting micro-orm. This and pagination.

 One week later, I've got SqlFu ready and I must say I'm really proud of it. It does all the stuff at top speed (as fast as peta poco and dapper). Only it has more features and it's very easy to plug in your own mapping code for those edge cases. You can find the source and the docs, along with some benchmarks on [GitHub](https://github.com/sapiens/SqlFu).


