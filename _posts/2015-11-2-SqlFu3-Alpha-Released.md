---
layout: post
title: Itroducing SqlFu 3
category: Sqlfu
---

I'm happy to report that SqlFu 3 finally got to a stage where it can be publicly usable. Although an alpha, it's more of a pre-beta, as the features I wanted are all in. But things can change, so that's why it's still an alpha for now.

**Warning!** Version 3 is NOT backwards compatible.

## It's not an ORM
At least in SqlFu case, "micro-ORM" is the wrong name. SqlFu is a data mapper (maps data to commands and rows to objects) with productivity helpers. While the objective of an ORM is to pretend Sql doesn't exist and we're all having fun using objects, in SqlFu we think sql and we're writing sql, but many of the most common operations are provided as strongly typed helpers (which are NOT Linq, but specialised string builders). A bit of abstraction exists, but it's merely a side effect, not an objective.

Objects are just data _source_ or data _destination_.

## Designed for versatility
I've learned a lot from the previous versions, issues posted by users and my own usage especially in a cloud environment. It's not about adding gimmicky stuff, but actual value that make things easier. For example, two of the new features are: support for working with multiple databases and transient error resilience. Nothing fancy, but it cuts down repetitive boiler plate code.

### New Features
* Multi database support
* Transient error resilience
* Strongly typed builders to create a table or to build dynamic queries for a table/view
* Sync/async support for any operation
* Any helper command can be customized
* SProc support (command and queries)
* Mapping to anonymous objects
* Faster mapping to dynamic

## Usage

Obviously, get the pre-release version from [nuget](https://www.nuget.org/packages/SqlFu/3.0.0-alpha-201511021041). Only SqlServer 2012+ (including Azure) is supported.

```csharp

//app startup
SqlFuManager.Configure(c =>
           {
               c.AddProfile(SqlServer2012Provider.Instance,"[connection string]");            
           });

//factory, register it in your Di Container
var getDb=SqlFuManager.GetDbFactory();

//pocos
 public class SomePost
     {
         public int Id { get; set; }
         public string Title { get; set; }
         public string Content { get; set; }
         public SomeEnum State { get; set; }
         public DateTime CreatedOn { get; set; }
     }

     public class Category
       {
           public string Name { get; set; }
           public int PostId { get; set; }
       }

//anywhere you need it, we inject a factory, NOT a connection
class SomeService{
   public SomeService(IDbFactory getDb){ }

}

//let's generate a create table statement. This is NOT mapping, just a builder
// only default types are abstracted.

using(var db=_getDb.Create()){
  db.CreateTableFrom<SomePost>(cfg =>
               {
                   cfg.TableName("Posts");

                   cfg.Column(col => col.Id, c =>
                   {
                       c.AutoIncrement();
                   })
                       .Column(col => col.Content, c =>
                       {
                           c.HasSize(75).Null();
                       })
                       .Column(c => c.State, c =>
                       {
                           c.DbTypeIs("nvarchar").HasSize("50").NotNull();
                       })
                       .Column(c => c.Title, c => c.HasSize(75))
                       .Column(c=>c.CreatedOn,c=>c.DbTypeIs("datetime2"))
                       ;

                   cfg.PrimaryKey(c=>c.OnColumns(t=>t.Id));
                   cfg.IfTableExists(IfTableExists.DropIt);
               });
   db.CreateTableFrom<Category>(cfg =>
               {
                   cfg.Column(c => c.Name, c => c.HasSize(70))
                       .ForeignKeyFrom<SomePost>(
                           c =>
                               c.Columns(e => e.PostId)
                                   .Reference(f => f.Id)
                                   .OnDelete(ForeignKeyRelationCascade.Cascade));
               });
}

//want to execute a create table as a task?
public class Creator : ATypedStorageCreator<Category> //implements ICreateStorage
    {
        public Creator(IDbFactory db) : base(db)
        {
        }

        protected override void Configure(IConfigureTable<Category> cfg)
        {
            cfg.Column(c => c.Name, c => c.HasSize(70))
                       .ForeignKeyFrom<SomePost>(
                           c =>
                               c.Columns(e => e.PostId)
                                   .Reference(f => f.Id)
                                   .OnDelete(ForeignKeyRelationCascade.Cascade));
        }
    }

//register all types implementing ICreateStorage into a di container
// then invoke all storage creators
//using autofac
_scope.Resolve<IEnumerable<ICreateStorage>>().OrderByAttribute().ForEach(s=>
            s.Create());        

//transient errors resilience, if connection times out or max number of connection is reached
//operation is automatically retried for 10 times with a delay between retries
// works with the db factory only (not connection)
_getDb.Do(db=> {
   //queries
   });
```

### Operations example

```csharp

  var func = db.GetDbFunctions();

 //build query but map only the first column
  db.GetQueryValue(t => t.From<SomePost>().Where(d => d.State == SomeEnum.Last).AllColumns());

 //build query from SomePost, select all columns and map them to SomePost
  db.GetSingle(d => d.From<SomePost>().Select(c => new SomePost()));

//simple insert
  var id=db.Insert(new SomePost() {Title = "my title"}).GetInsertedId<int>();

//custom insert
db.Insert(new {Id = 1, Name = "foo"}, c =>
                {
                    c.TableName="myTable";
                    c.CmdOptions = cmd => cmd.CommandTimeout = 500;
                });

 // select  count([Id]) from [Posts]
 //build query but the outcome should be a different type
  var all = db.GetQueryValue(f => f.From<SomePost>().Select(d => func.Count(d.Id)).MapTo<string>());

//select  count([Id]) as Total from [Posts]
//group by [CreatedOn]
//order by [CreatedOn]
//OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY
 var rez= db.Query(f => f.From<SomePost>().GroupBy(d=>d.CreatedOn).OrderBy(d=>d.CreatedOn).Limit(10,0).Select(d => new {Total = (int)func.Count(d.Id)}));

 //simple update
  db.Update<SomePost>().Set(d => d.Title, "changed").Where(d => d.Id == id).Execute().Should().Be(1);

//anonymous object update
  db.Update(o =>
  {
      o.TableName = "Posts";
      o.DbSchema = "dbo";
  }, d => d.FromData(new {Id = id, Title = "other"}).Ignore(i=>i.Id)).Where(d => d.Id == id).Execute();

 //simple pagination
  var paged = db.FetchPaged<SomePost>(c => c.Sql("select * from Posts"),new Pagination());

  db.DeleteFrom<SomePost>(d => d.Id == id);

//stored procedure
var result=db.QuerySProc<dynamic>("spTest", new {inputParam = 46, _outParam = ""});
var foo=result.OutputValues.outParam;


//value object mapping
//email is automatically converted to string when insert/update and from string to email when select
SqlFuManager.Config.MapValueObject<Email>(/* from*/ e=>e.Value,/* to */o=>new Email(o.ToString()));
db.Insert(new EmailStore() {Data = new Email("test@bla.com")});

```

This is a very quick preview of SqlFu 3, I hope I'll have the time to write some proper docs. In the mean time feel free to play with it and to [report any bugs](https://github.com/sapiens/SqlFu/issues).
