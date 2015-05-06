---
layout: post
title: Asp.Net Mvc Apply Filters By Convention
category: Caveman Tools
---

After using conventions to define routes, how about using conventions to apply filters? I mean, I have to decorate every controller in the Admin section with at least 2 attributes: [RequirePermission(UserRights.Login)] and [OverrideTheme("admin")]. I'm using the Handler convention so basically I have as many controllers as I have actions. I also want to enable [[SmartAction]](http://www.sapiensworks.com/blog/post/2013/01/09/DRY-AspNet-Mvc-Controllers.aspx) for all POSTs and to have some specialized Ajax Only Controllers.

 The default way is to clutter each action (or controller) with a stack of attributes or worse, to have a base Controller with these attributes and the inherit from it. I consider the inheritance solution to be a poor one, this is one of the cases where you shouldn't use it.

 We need a better way and luckily, it's trivial to implement one. The idea is very simple: I want to apply a filter according to a convention. Of course, I start with an interface to define a filter policy. All the filters are considered to be global filters.

  
```csharp
public interface IFilterPolicy
    {
        int? Order { get; }
        /// <summary>
        /// Filter instance
        /// </summary>
        object Instance { get; }
        
        bool Match(MethodInfo action);
    }
```
  This policy is then used by a filter provider to know when to return the filter. As I've said, it's trivial.

  
```csharp
internal class FiltersWithPoliciesProvider:IFilterProvider
    {
        Dictionary<string,IEnumerable<Filter>> _filters=new Dictionary<string, IEnumerable<Filter>>();
        public FiltersWithPoliciesProvider(IEnumerable<FilterActionInfo> filters)
        {
            foreach (var filter in filters)
            {
                _filters.Add(filter.Key,filter.Filters);
            }
        }
        
        
               public IEnumerable<Filter> GetFilters(ControllerContext controllerContext, ActionDescriptor actionDescriptor)
        {
            var key = FilterActionInfo.CreateKey(actionDescriptor.ControllerDescriptor.ControllerType,
                                                 actionDescriptor.ActionName);
            IEnumerable<Filter> filters;
            if (!_filters.TryGetValue(key, out filters))
            {
                filters=new Filter[0];
            }
            return filters;
        }
    }
```
  For a quick and elegant setup

  
```csharp
//setup DI container
    builder.RegisterAssemblyTypes(ThisAssembly).Where(t => t.DerivesFrom<IFilterPolicy>()).AsSelf();

   //tell FiltersPolicy where to find actions and policies
     FiltersPolicy.Config.RegisterControllers(asm);
     FiltersPolicy.Config.RegisterPolicies(asm);
     
     //tell FilterPolicy to build and register the provider
     FiltersPolicy.Config.RegisterProvider(FilterProviders.Providers);
```
  Now let's write some conventions

  
```csharp
//All admin views are using the Admin theme
 public class AdminThemeIsFixed:IFilterPolicy
    {
        public AdminThemeIsFixed()
        {
            Instance = new OverrideThemeAttribute("admin");
        }
        public bool Match(MethodInfo action)
        {
            return action.DeclaringType.Namespace.Contains("Admin");
        }

        public int? Order { get; private set; }

        /// <summary>
        /// Filter instance
        /// </summary>
        public object Instance { get; private set; }
    }
 
 //All controllers from Admin must be authorized
  public class AllAdminActionsRequiresLogin:IFilterPolicy
    {
        public AllAdminActionsRequiresLogin()
        {
            Instance = new RequiresPermissionAttribute(UserRight.Login);
        }
        public bool Match(MethodInfo action)
        {
            return action.DeclaringType.Namespace.Contains("Admin");
        }

        public int? Order { get; private set; }

        /// <summary>
        /// Filter instance
        /// </summary>
        public object Instance { get; private set; }
    }
    
    //All actions from controllers having ajax in name are constraint to ajax requests
  public class AjaxControllersConvention:IFilterPolicy
    {
        public AjaxControllersConvention()
        {
            Instance = new AjaxAttribute();
        }
        public bool Match(MethodInfo action)
        {
            return action.DeclaringType.Name.Contains("Ajax");
        }

        public int? Order { get; private set; }

        /// <summary>
        /// Filter instance
        /// </summary>
        public object Instance { get; private set; }
    }
```
  There you have it, the easiest way to apply filters selectively via conventions. Now, your controllers are less cluttered and you don't have to remember all the attributes or to misuse inheritance.  
   
 Note that this doesn't interfere with the global asp.net mvc filters, so you can register some filters to apply to every action while other are applied according to a convention. You can check out the [whole source here](https://bitbucket.org/sapiensworks/caveman-tools/src/fe708f2cfe56f4f092a41f222ff887eda238dc3d/src/CavemanTools.MVC/Filters?at=devel).


