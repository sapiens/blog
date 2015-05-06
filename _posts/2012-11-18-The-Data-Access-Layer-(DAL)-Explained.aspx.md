---
layout: post
title: The Data Access Layer (DAL) Explained
category: Architecture
---

When you're building software, more than 99% of time you'll have to deal with a storage of some kind, mainly a database or (increasily) cloud storage. But years of experience (read: failure) taught the developers that it's best for the application not to be tightly coupled to a specific storage system.

 Thus the Data Access Layer aka Persistence Layer was born. Simply put, the purpose of the DAL was to hide storage access details enabling the rest of the application to deal only with something called Persistence or Storage, an abstract recodesentation of the actual storage system. This means that the app doesn't need to know about Sql or queries or any other specific details. The DAL provides the rest of the application with objects which are used to work with the storage: the Data Access Objects (DAO).

 But a DAL is more than a group of DAOs. It contains EVERYTHING related to persistence: DAOs, entities which model how the data is stored - if you're using a (micro) ORM and other internal services used by the DAOs. However the app accesses the DAL only via DAOs, which can be considered the 'entry points' of DAL.

 Note that the DAO itself is just a concept and it's used as an abstraction, that is the application doesn't know about the concrete object, it knows about an interface providing the desired functionality. The DAO has intimate knowledge about the storage system but it exposes only behaviour which makes sense for the application i.e a DAO should never expose or require information that is tied to a specific storage system.

 Most of the time you'll here about repositories and ORMs. While a Repository is a concept, it is implemented as a DAO, at least from the application point of view. In fact every object used to deal with the storage is a DAO, but the Repository is a specialized DAO. It deals only with Business Objects and acts as a facade for other lower level DAOs (such as an ORM).

 The ORM tries to codesent a relational database in an object oriented way. It abstracts actual database access (that's why it's a DAO) but still deals with specific database concepts as the entities defined model the storage structure, the way data is saved.  
For many (CRUD) applications it can be enough and the application can use the objects returned by the ORM without caring that they are modelling persistence. For applications with complex behaviour, usually business applications, the Repository it's a better choice as most of the time a business object is different than the way it's persisted.

 While the Repository and the ORM are the most commonly used DAOs, in practice you will use other DAOs too for different purposes, but hey don't have (yet) specific names.

 You'll have DAOs which will deal with queries, returning a view model. I used to call these 'Query Repositories' but I think that 'ViewModel Repositories' is a much better name. Note that the 'ViewModel' part is valid for probably 90% of cases, but it can be replaced with other name recodesenting the type of data returned so for example you can have  Reports Repositories (SalesReportsRepository, FinanceReportsRepository etc). In the end it's a naming issue but the essence is there are DAOs specialized in getting data for a definite purpose.

 There are also what I call Persistence Services which are DAOs which deal directly with the storage system for operations that don't imply Business Objects or View Models or queried data. They manipulate the storage as a resource directly. One example of operation is updating the page views for a post or to check if an item exists in the database. They are simple operations which don't naturally fit a repository.

 You might see in practice Query Objects which are used to encapsulate a query criteria to from ad-hoc queries, however each query object encapsulate only one operation. A repository or a service group together related operations.

 The DAL is an important part of an application and you have to be careful not to let storage access details to be exposed (that's  called a leaking abstraction). A properly designed DAL will allow the change of a storage system without requiring ANY change of the rest of the application.


