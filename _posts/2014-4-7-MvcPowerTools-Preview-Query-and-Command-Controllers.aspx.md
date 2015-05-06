---
layout: post
title: MvcPowerTools Preview: Query and Command Controllers
category: MvcPowerTools
---

[MPT](https://github.com/sapiens/MvcPowerTools) gives you a helping hand if you want to use [Query](http://lostechies.com/jimmybogard/2013/10/29/put-your-controllers-on-a-diet-gets-and-queries/) or [Command Handlers](http://lostechies.com/jimmybogard/2013/12/19/put-your-controllers-on-a-diet-posts-and-commands/)  . While not exactly the same implementation as Jimmy's, it's the same idea, only combined with the Handler Controller approach (in a nutshell, the Controller is the action and its methods handle GET/POST, very similar to WebApi or REST approach). It also makes use of the One Model In, One Model Out convention.

 Note that these are opinionated choices, alternatives and not yet another silver bullet solution. Just another way to do things, that might suit your style. Enough said, let's see some code (pasted directly from an app I did recently, and yes, that's ALL the controller and handler code).

  
```csharp
public class TagsListController : QueryController<PagedInput, TagsListModel>
    {
        public override ActionResult Get(PagedInput input)
        {
            return Handle(input);
        }
    }

 public class TagsListHandler : IHandleQuery<PagedInput, TagsListModel>
    {
        private readonly IQueryTaxonomyStats _stats;

        public TagsListHandler(IQueryTaxonomyStats stats)
        {
            _stats = stats;
        }

        public TagsListModel Handle(PagedInput input)
        {
            var model = new TagsListModel();
            model.Page = input.Page;
            model.Resources = _stats.GetTagList(input.ToPagination());            
            return model;
        }
    }
```
  Let's see: you have an input model _(PagedInput_) received by the controller. The controller just asks the base _QueryController_ to handle it, in essence, to return the view (output)model for that input. All that the query controller does is to use the Dependency Resolver to invoke the defined handler that accepts _PagedInput_ and returns _TagsListModel_. In this specific scenario I'm using a repository, but I could have just used an ORM directly. The handler's job is to assemble the view model and here it's quite trivial.

 You may ask yourself why should we bother with this when we could've put that handler behaviour directly in the controller. The answer is: decoupling. _TagsListHandler_ is somewhere outside asp.net mvc, UI, in my app is part of the QueryModel project . Also, as you can see, it doesn't depend on asp.net, controllers, HttpContext etc. It's totally decoupled from the UI and this means it easy to test and it can be reused outside asp.net mvc.

 So, _TagsListController_'s job is simply to build the input model (in this case it doesn't have to do anything) that will be use by the query handler. And all controller handling queries look like this one. The important part is the query handler itself, which is just a class from a querying layer.

 Now let's see a command controller.

  
```csharp
public class AddInformationsController : CommandController<AddInformationsModel,NoResult>
    {

        public ActionResult Get()
        {
            var model = new AddInformationsModel();
            return View(model);
        }

        public override ActionResult Post(AddInformationsModel model)
        {
            return Handle(model,
                d =>
                    this.RedirectToController<BrowseResourceController>(
                        c => c.Get(new BrowseResourceInput() {Type = ResourceType.Informations})));           
        }
	}
	
	 public class AddInformationsHandler:IHandleCommand<AddInformationsModel,NoResult>
    {
        private readonly IDispatchMessages _bus;

        public AddInformationsHandler(IDispatchMessages bus)
        {
            _bus = bus;
        }

        public NoResult Handle(AddInformationsModel model)
        {
            //map model to domain entity
			//add entity to storage
			//publish domain event
            return new NoResult();
        }
    }
```
  So, I want to add an information. The Get method returns a view with an empty view model. Then, the form is submitted and the Post method is invoked. The command controller will use the DependencyResolver to request and execute a handler which takes an_ AddInformationsModel_ input (command) and returns _NoResult_ (if you want to return let's say an id, define a class SomethingId encapsulating the id and use that instead of NoResult). Please excuse the pseudocode comments, the actual code is irrelevant here. The controller's _Handle_ method second argument is a lambda for what to do after the command is handled successfully, in this case a redirect. Again, the advantage is that we keep the command handler outside of UI, decoupled from asp.net mvc. In this example, the handler is part of the Domain (I don't have a Services layer).

 As you can see, the controllers are as slim as they can get (1-2 lines of code) while the real logic is in the proper layer. And if you followed closely, you could see a convention at work here:  _TagsList_**Controller**, _TagsList_**Model**, _TagsList_**Handler**. If the input model had more then a 'page' property, I would have had a _TagsList_**Input** class as well. It's easy, even now, after a few good months to go directly to the class I need, when I want to change something.

 A final note, this approach really require the use of a DI Container and of course, the handlers need to be registered with one.


