---
layout: post
title: How To Design Easily Testable Asp.Net Mvc Action Filters
category: Best Practices
---

While Asp.Net Mvc is a much more test friendly framework compared to WebForms, some things are still complicated to test. Custom Action Filters are very common and naturally you want to test them too. But you still have to setup a whole chain of dependencies (the filterContext argument) even for trivial filters.

 However, we can test things much easier, without the complicating setup. Let's say I want a filter that takes the id from the url, gets the user for that id and then cache it in HttpContext.Items. It looks something like this

  
```csharp
public class MyFilter:IActionFilter
    {
        
        public void OnActionExecuting(ActionExecutingContext filterContext)
        {
            var repo = DependencyResolver.Current.GetService<IMyRepository>();
            int uid;
            if (!int.TryParse(filterContext.RouteData.GetRequiredString("id"),out uid))
            {
                filterContext.Result=new HttpNotFoundResult();
                return;
            }
            var user = repo.GetById(uid);
            if (user == null)
            {
                filterContext.Result = new HttpNotFoundResult();
                return;
            }
            filterContext.HttpContext.Items["user"] = user;
        }
        
     
        public void OnActionExecuted(ActionExecutedContext filterContext)
        {
            
        }
    }
```
  While the behavior is quite simplistic look what I have to fake: the DependecyResolver, the RouteData and the HttpContext. Let's refactor it so that it's easily testable

  
```csharp
public void OnActionExecuting(ActionExecutingContext filterContext)
        {
            var repo = DependencyResolver.Current.GetService<IMyRepository>();            
            var user = GetUser(repo, filterContext.RouteData.GetRequiredString("id"));
            filterContext.Result = CacheUser(user, filterContext.HttpContext.Items);                        
        }

        public User GetUser(IMyRepository repo, string id)
        {
            int uid;
            User user = null;
            if (int.TryParse(id, out uid))
            {
                user = repo.GetById(uid);                
            }
            return user;
        }

        public ActionResult CacheUser(User user, IDictionary cache)
        {
            if (user == null)
            {
                return new HttpNotFoundResult();
            }
            cache["user"] = user;
            return null;
        }
```
  I've isolated the desired behavior in 2 public easily testable methods. With _GetUser_ you can test what happens if the 'id' is or not a valid user id having only to mock **IMyRepository**. With _CacheUser_ you can test what happens if the **User** is null or not. None of these methods depend on asp.net mvc so no need to fake all the **ActionExecutingContext** object. Better yet, the behavior can be used outside the action filter as well.

 It's a trivial example because it's a simple technique: extract the actual behavior in one or more relevant methods which take only the dependencies they need. In this example we needed only the repository and an **IDictionary**. Very rarely you'll need HttpRequest or HttpContext as a dependency, usually you need only a property.

 Here's another example. I want to allow access only to users with the email belonging to some domain.

  
```csharp
public class DomainEmailAuth:AuthorizeAttribute
    {
        public string EmailDomain { get; set; }

        protected override bool AuthorizeCore(System.Web.HttpContextBase httpContext)
        {
            if (base.AuthorizeCore(httpContext))
            {
                var repo = DependencyResolver.Current.GetService<IUserServices>();
                return IsAuthorized(repo, httpContext.User.Identity.Name);
            }
            return false;
        }

        public bool IsAuthorized(IUserServices users,string username)
        {
            Email email = users.GetEmail(username);
            if (email != null)
            {
                if (email.BelongsToDomain(EmailDomain))
                {
                    return true;
                }
            }
            return false;
        }
    }
```
  The relevant behavior needs only the **UserServices** and the username in order to get the email and find out if it is allowed. It can be tested without caring about asp.net or the AuthorizeAttribute itself.


