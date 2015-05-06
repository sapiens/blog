---
layout: post
title: StartItUp 1.2.0 Released
category: .Net
---

I love using this little utility in every app. After almost 2 years I finally added a way to specify app config tasks without using an attribute. Now my config classes look like this

  
```csharp
public class ConfigTask_0_Logging {
public static void Run();
}

public class ConfigTask_1_Container {}

public class ConfigTask_Routing {}

public class ConfigTask_ViewEngines {} 

//etc
```
  I think it's obvious what they do. And my Global.asax looks like this

  
```csharp
protected void Application_Start()
{
     StartupTasks.Run();                      
}
```
  Yeah, I like 1 line of code methods. Self-explanatory too.

 P.S: I know about the Bootstrapper library. It does way too much for my taste. De gustibus non est disputandum .


