---
layout: post
title: Introducing SqlFu 1.2.0
category: SqlFu
---

1.2.0 is a milestone for SqlFu as it brings new features to the table (besides the usual bug fixing and minor improvements)

  
  * **Database Tools**  
    Common table management got much easier. You can rename, drop, truncate or check table existence using C# only. Simple but very useful.  
  * **Database Migrations**  
    You know the fun when you have to upgrade db schema, even more so when you have to support more than one database engine. Well, things will be much more easier with SqlFu. Just define migration tasks, put an attribute on them and let the Automatic Migrator keep track of everything. OF course, you still have to write the actually SQl (or maybe not - check the next feature :) ) to modify the db structure, but the whole logistics is taken care of. You just concentrate on the actual Sql stuff. 
  * **Fluent DDL Builder**  
    Or 'How to write create/alter table scripts using only C#' . Supported databases are SqlServer, Mysql, Postgres and Sqlite (only partially). You can write common Sql as well as customize the script for each supported db. All C# with code completition. 
  * **Stored Procedure support**  
    Support for input and output parameters.   You can read about these features in detail on the [SqlFu Wiki](https://github.com/sapiens/SqlFu/wiki)


