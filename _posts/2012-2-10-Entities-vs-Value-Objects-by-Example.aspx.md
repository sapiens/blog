---
layout: post
title: Entities vs Value Objects by Example
category: Domain Driven Design
---

**Rule of thumb**: _Entities_ have an unique identity and one entity can't be substituted for another just because they have the same data. _Value Objects_ have no identity and they can substitute each other if they have the same data. Any change to a value object means a new value object is born. An object can be an entity or a value object depending on the context.

 **A post, a comment or a product** are entities because  they have an id and changing some of their properties doesn't change the resource we're working with. It's one of the reasons you work with ids instead of the whole objects.

 **A tag** is a value object. Change the value of the tag and the tag changes as well, becoming a different tag.

 **A category** is an entity. Changing the category name doesn't change the category itself. All the resources from the category still point to the same category.

 **A coin** is a value object. You can substitute one coin with another providing they have the same purchasing power.

 **A lucky coin** is an entity. You care about that coin in particular and you can not substitute it with another coin.


