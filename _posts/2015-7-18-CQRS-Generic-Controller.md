---
layout: post
title:  Make Your CQRS Web App More Maintainable With a Generic Controller
category: Asp.Net
---

From a design point of view if you want a proper separation of concerns (SoC) in a web app, it looks like this
* You have a controller/action "in charge" of a http request
* The controller maps the request to a command (input data)
* The controller invokes an application service passing the input data. In a CQRS app, you might have a command send to a command handler, which is the implementation of an application use case, ergo an application service.
* The service updates the domain objects then persists them. Domain events are published if you're using the domain events pattern.
* The service returns a result which is mapped by the controller to a http response.

You want to keep the controllers very slim because their purpose is just to act as adapter between the web and the application services. The actual app behaviour is in the application service. So for each use case we'd have an application service and a controller/action which will invoke it. And this can become very boring quickly.

## Problem: Repetitive boilerplate code required to respect SoC

Some devs will consider that DRY is a good reason to violate SoC so they merge a controller with an application service. Everything that should be in that service is put in a controller. While it works it does have a big drawback: it's so easy to couple yourself to the MVC framework. Add an ORM into the mix and the controller becomes a part of persistence as well. This is more prevalent for Query controllers.

While _some_ apps are trivial enough that we don't need to bother with abstractions and SOLID principles, if you actually build a business app or at least some app that will evolve in time and you want it maintainable, the above "pragmatic" solution will slowly lead to a ball of mud.

We want SoC but we don't want to have hundreds of repetitive controllers with 1-2 lines that in the end amounts to "call some service" and return a result.

## Solution: Enter the Generic Controller

What we want is to have one controller where to put all the repetitive bits (validation,specific authorization, common errors scenario handling) that will act as a "host" for any app service (the Command part of CQRS) or query handlers (the Query part of CQRS).Yes, one controller to rule'm all. Or several, because you want specific customizations for some parts of your app.

In a nutshell it look like this (although this list is common for both command/query there would be a specific implementation for each case):

* Controller is invoked for a route pattern which captures a command/query name
* Based on the name (or other convention), the request is mapped to the corresponding model (a command/query) i.e _AddUser_ or _GetUser_ . Note that these are inputs, _not_ handlers.
* For commands basic validation can be performed
* Other common stuff, like ensuring the user is authenticated (if you need it) or populating the model with common information like the user id
* Use case specific behaviour. This is basically aspect oriented programming. We annotate the model with attributes that will trigger specific behavour in the controller. An example is to check if the curent user is Admin or has the _CanManageUsers_ permission. Or that the model expects a certain property populated.
* Based on the input model, a service/query handler is selected then executed and a result is returned.
* A non-null result is then returned as Json (for a Api) or a view model.
* A convention can say that if a result is null then return a 406 status (Not acceptable) for a Api or some error page.

### Implementation

Be aware that the implementation varies according to your app's needs. The only thing generic (sic) enough is the principle, the code needs to be adapted to your situation. This means the example below is not meant to be copy/pasted, it just shows you a concrete implementation. The code is an excerpt from the project I'm working on (while my project uses [NancyFx](http://nancyfx.org/) I'm gonna show you asp.net webapi/mvc implementations).

But first, let's see what we have and what we need. I'm using CQRS all the way up (or down) so my application services are command handlers while querying is done by query handlers (which work directly with the db). For both my convention is to have one _unique_ model in, one model out. I also have specific interfaces for each case

```csharp
class AddUserService:IHandleCommand<AddUser,CommandResult>
{
    public CommandResult Handle(AddUser cmd){ }
}

class GetUsersQuery:IHandleQueryAsync<GetUser,UserInfo[]>
{
  public Task<UserInfo[]> Handle(GetUser query){ }
}

```
Each service handles one or more related commands, the name isn't important but the abstraction is, because this is how we'll ask the DI Container the service/query handler we need. But remember that in our controller we only have a name extracted from the url. We need a way to map that name to our input and to detect the correct service that will handle the command/query.

For that, I've come up with the following class (this needs to be adapted to your specific needs)

```csharp
public class ServiceHandlersCache
  {
      Dictionary<string,ServiceHandlerItem> _items=new Dictionary<string, ServiceHandlerItem>();

      //scan and register the input/result models from the found command/query handlers
      public void AddHandlers(Assembly asm)
      {
          var types = asm.GetPublicTypes(t => !t.IsAbstract && (t.ImplementsGenericInterface(typeof(IHandleCommand<,>)) || t.ImplementsGenericInterface(typeof(IHandleQueryAsync<,>)))).ToArray();
          asm.GetTypesDerivedFrom<RequestInput>()
              .ForEach(t =>
              {
                  var result = FindResultModel(t, types);
                  if (result==null) return;
                  var item=new ServiceHandlerItem(t,result);
                  _items.Add(item.CmdId,item);
              });

      }

      public ServiceHandlerItem GetModels(string cmdId)
      {
          return _items.GetValueOrDefault(cmdId);
      }

      static Type FindResultModel(Type input,Type[] types)
      {
          var result = types.Select(t =>
          {
              var interAll =
                  t.GetInterfaces()
                      .Where(i => i.Name.StartsWith("IHandleCommand") || i.Name.StartsWith("IHandleQueryAsync"));
              var inter = interAll.FirstOrDefault(i => i.GetGenericArgument() == input);
              if (inter==null) return null;
              return inter.GetGenericArgument(1);
          }).FirstOrDefault(d=>d!=null);
          return result;
      }
  }

  public class ServiceHandlerItem
   {
       public Type Input { get; set; }
       public Type ResultModel { get; set; }

       public string CmdId
       {
           get { return Input.Name; }
       }

       public RequestInput CreateInputInstance()
       {
           return Input.CreateInstance() as RequestInput;
       }

       public AccountRights[] RequiredRights { get; private set; }

       public ServiceHandlerItem(Type input, Type resultModel)
       {
           Input = input;
           RequiredRights=input.GetAttributeValue<RequiresAttribute, AccountRights[]>(a => a.Right);
           ResultModel = resultModel;
       }
   }
```

`ServiceHandlersCache`, which is used as a singleton, maintains a cache of the input/result models which is injected into the controller. Given an identifier it returns the input/result models that will be used by the controller. While I'm using this mainly for Api purposes, it can be easily adapted to web sites.

```csharp

[Requires(AccountRights.ManageUsers)]
public class AddUser:RequestInput
{

}

 //controller constructor
  public GenericController(ServiceHandlersCache cache) {}


//actions
  [Route("api/{cmdId}")]
  [HttpPost]
  [Authorize]
public CommandResult Post(string cmdId)  
{
   var models=_cache.GetModels(cmdId);

  //handle null scenario {}

   var cmd=models.CreateInputInstance();

   this.TryUpdateModel((dynamic)cmd);

   if (!ModelState.IsValid){
     var result=ModelState.ToCommandResult();//ext method
     return result;
   }

  //ext method to check for specific AccountRights if the model has it
    if (!this.HasAnyOfRights(models.RequiredRights)){
    return null;
  };

  var result=cmd.ExecuteAndReturn(models.ResultModel) as CommandResult;
  return result;
}
```

`ExecuteAsyncAndReturn` is an [extension method](https://bitbucket.org/sapiensworks/caveman-tools/src/19449cd78e4e51fea07f6fd923f9b8ae75574606/src/CavemanTools/Infrastructure/MessagingMediator.cs?at=default) that uses the Di Container to create the service implementing `IHandleCommand<Input,Output>` then executes it returning the result.

For query handlers we have a similar action

```csharp

  [Route("api/{queryId}")]
  [HttpGet]
public Task<object> Get(string queryId)
{
  var models = _cache.GetModels(queryid);

  if (!this.HasAnyOfRights(models.RequiredRights)){
    return null;
  };

    var query = models.CreateInputInstance();

    var result = await cmd.QueryAsyncTo(models.ResultModel, cancel);

    return result;  
}
```

I'm using POSTs to identify commands and GETs to identify queries. We can create additional attributes to annotate the input model and based on them, invoke a specific behaviour in our controller. Those behaviours are basically decorators that need to be generic enough to be reusable. Until now, in my app I didn't need more than permission checking.

The point here is to extract the repetitive infrastructural behaviour into a controller and action filters. This allows us to focus only on the actual app behaviour. Adding a new use case means adding a new command, handler, validator i.e all the new things required, but not the boilerplate stuff like controllers/actions. Instead of having 10 services, 15 queries and 25 controller actions, I now have 10 services, 15 queries and 2 controller actions. Less code, DRY code.

Of course, the great benefit here is decoupling of the app service from the api/mvc framework. Our app service knows _only_ about its input (nothing http or UI framework related) and the controller just prepares that input from a request. If the service needs a user id or an Ip, the input model can be decorated with an attribute that will tell the controller to populate that specific property from the model.

Note that I'm not using a service bus in this example although I am using one in my app, but that's a special case that I'll handle (pun intended) in the next post.
