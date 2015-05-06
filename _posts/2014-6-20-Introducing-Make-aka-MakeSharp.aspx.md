---
layout: post
title: Introducing Make# aka MakeSharp
category: .Net
---

I don't always have many needs when using a build automation tool (with C# as the scripting language) but when I do, I make a mess with the procedural approach. Static methods and global variables simply don't work if you want a tidy, reusable and flexible functionality. At least in my case it doesn't work, since I'm so used with the object oriented mindset and I always feel the need to define a class and extract some things into private methods etc.

 So, I've decided to take my [CSake (C# Make)](https://github.com/sapiens/csake) project to the next level i.e I've rewrote it to be more compatible with the OOP mindset and to use the (not so) new [scriptcs](https://github.com/scriptcs/scriptcs) that everyone's raving about. Enter Make#(MakeSharp) the build automation tool where you can use C# in an OOP way (like God intended). And because I do like intellisense, all of my projects using Make# also have a Build project referencing Make# executable and helper library where I can code my build 'script' with all the VS goodness. And the script is in fact a bunch of classes (you can almost call it an app) which do the actual work.

 You can read details and a generic example on the [Make# github page](https://github.com/sapiens/MakeSharp) but in this post I want to show a bit more advanced usage. This is the build script for a project of mine called [MvcPowerTools](https://github.com/sapiens/MvcPowerTools) (well, in fact there are 2 because it contains the WebApi tools, too). My needs are:

  
  * clean/build solution; 
  * update nuspec files with the current version, as well as versions for some deps. One dependency is a generic library I'm maintaining and evolving and many times I'd need some features which turn out to be generic enough to be part of that library. And since I don't want to make 3 releases a day for that library, I'm using it directly, hence its version it's not stable. So, I want to update the dep version automatically; 
  * build packages as code release, this means the project's version in nuspec would have the "-code" suffix; 
  * push packages.  I need all this steps for both projects and I want my script to allow me to specify I want only one or both projects to be built. This is the script (you can see it as [one file here](https://github.com/sapiens/MvcPowerTools/blob/master/build/build.cs)) .

  
```csharp
public class PowerToolsInit : IScriptParams
{
    public PowerToolsInit()
    {
        ScriptParams=new Dictionary<int, string>();
		Solution.FileName = @"..\src\MvcPowerTools.sln";
    }

    List<Project> _projects=new List<Project>();

    public IEnumerable<Project> GetProjects()
    {
        if (_projects.Count == 0)
        {
            bool mvc = ScriptParams.Values.Contains("mvc");
            bool api = ScriptParams.Values.Contains("api");
            bool all = !mvc && !api;
            if (mvc || all)
            {
                _projects.Add(new Project("MvcPowerTools",Solution.Instance){ReleaseDirOffset = "net45"});
            }
            if (api || all)
            {
                _projects.Add(new Project("WebApiPowerTools",Solution.Instance){ReleaseDirOffset = "net45"});
            }
        }
        return _projects;
    }

    public IDictionary<int, string> ScriptParams { get; private set; }
}
```
  You see that I'm defining a class using init data: **PowerToolsInit**. This is how I override the default implementation of **IScriptParams**, I'm providing my own. In this class I'm deciding which projects are to be built based on script arguments. **Solution**  and **Project** are codedefined helpers of Make# (intellisense really simplifies your work ). I have only one solution so I'll be using it as a singleton.

  
```csharp
public class clean
 {
     public void Run()
     {
        
         BuildScript.TempDirectory.CleanupDir();
        Solution.Instance.FilePath.MsBuildClean();        
     }

 }

[Default]
[Depends("clean")]
public class build
{

    public void Run()
    {
        Solution.Instance.FilePath.MsBuildRelease();
    }
}
```
  Self explaining. **BuildScript** is another codedefined helper. _MsBuildClean_ and _MsBuildRelease_ are windows specific helpers (found in **MakeSharp.Windows.Helpers.dll** which comes with Make#) implemented as extension methods.  
  


  
```csharp
[Depends("build")]
public class pack
{
    public ITaskContext Context {get;set;}

	public void Run()	
    {

	    foreach (var project in Context.InitData.As<PowerToolsInit>().GetProjects())
	    {
	        "Packing {0} ".WriteInfo(project.Name); //another helper
            Pack(project);
	    }
      
    }
	
   void Pack(Project project)
    {
        var nuspec = BuildScript.GetNuspecFile(project.Name);
        nuspec.Metadata.Version = project.GetAssemblySemanticVersion("code");
	    
        var deps = new ExplicitDependencyVersion_(project);
        deps.UpdateDependencies(nuspec);
        
        var tempDir = BuildScript.GetProjectTempDirectory(project);
	    var projDir = Path.Combine(project.Solution.Directory, project.Name);
        var nupkg=nuspec.Save(tempDir).CreateNuget(projDir,tempDir);
	    Context.Data[project.Name+"pack"] = nupkg;
    }
}


class ExplicitDependencyVersion_
{
    private readonly Project _project;

    public ExplicitDependencyVersion_(Project project)
    {
        _project = project;
    }

    public void UpdateDependencies(NuSpecFile nuspec)
    {
         nuspec.Metadata.DependencySets[0].Dependencies.Where(d=>d.Version.Contains("replace"))
         .ForEach(d=> 
                d.Version=_project.ReleasePathForAssembly(d.Id+".dll").GetAssemblyVersion().ToString());
    }
}
```
  Now this is interesting. The **Context** property allows Make# to inject a codedefined **TaskContext** that can be used to access script arguments (or in this case the init object) and pass values to be used by other tasks. **BuildScript.GetNuspecFile** is a helper returning a **NuSpecFile** object (codedefined helper) which assumes there is a nuspec file with the project name available in the same directory as the build script. The **GetAssemblySemanticVersion** method of Project allows you to specify versioning details like code-release or build meta as defined by [Semver](http://semver.org). For Nuget purposes any text suffix marks the package as code-release.

 In order to update the dependencies version, I've created an utility class for that (default task discovery convention for Make# says that a class with the "_" suffix is not a task, i.e just a POCO) and my convention to indicate in a nuspec that a package dependency's version needs to be updated is that the version contains "replace", like this

  
```csharp
<dependency id="CavemanTools" version="0.0.0-replace" />
```
  Then I tell the **NuSpecFile** object to save the updated nuspec in the project's temp folder then I invoke the codedefined _CreateNuget_ helper. Then I save the returned nupgk file path into **Context.Data** so that it can be used by the next task.

  
```csharp
[Depends("pack")]
public class push
{
    public ITaskContext Context { get; set; }

    
    public void Run()
    {
        foreach (var project in Context.InitData.As<PowerToolsInit>().GetProjects())
	    {
	        var nupkg=Context.Data.GetValue<string>(project.Name+"pack");     
            BuildScript.NugetExePath.Exec("push", nupkg);
	    }
      
        
       
    }
}
```
  Push doesn't do much. It gets the nupkg paths from **Context.Data** then invokes the _Exec_ helper to execute nuget.exe to push the pakage. By default, NugetExePath assumes the nuget.exe is in the "..\src\.nuget" directory. You can change that.

 And here you have it: the implementation to build MvcPwerTools. I don't know about you, but I think this object oriented approach is much easier to maintain, rather than functions and global variables. [MakeSharp](https://github.com/sapiens/MakeSharp) is already available as a [nuget package](https://www.nuget.org/packages/MakeSharp/).


