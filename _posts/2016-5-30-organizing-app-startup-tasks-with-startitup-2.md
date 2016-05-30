---
layout: post
title: Organizing App Startup Tasks With StartItUp
category: .net
---

Every time I start a new project I go through the same ritual: configure logging, setup DI Container, configure data mapper, service bus, event store etc. Ok, that's not really a problem, 'cause I don't do that every day, but I do need to change some of this stuff often enough to require a more organized approach. 

For some time, I've had a small library that allowed me to organize the startup tasks into classes, but I needed a bit more flexibility: the ability to pass values/context between tasks without much ceremony. And this is how version 2 of `StartItUp` came to be. But let's see how it makes things easier.

Well, here's a self explaining screenshot

![Imgur](http://i.imgur.com/gZl3Lja.png)


All tasks are classes named according to the "ConfigTask_[order]_[TaskName]" convention.

`AppSettings' is the ... settings object that in most apps would be a static class, but I prefer an instance that I can register in the DI Container as a singleton if I need to inject it somewhere. It has options like email addresses, logging directory, db connection string etc. It's also used as a state to be shared by the config tasks.

```csharp

 public class AppSettings : StartupContext
    {
       
        public AppSettings(IAppBuilder owin)
        {
            ContextData["owin"] = owin;
            LogDirectory = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "App_Data") + Path.DirectorySeparatorChar;
#if DEBUG
            IsDebug = true;
#endif
        }

        public bool IsDebug { get; set; }

        public string DbConnection { get; set; } = LocalDb;

        public string LogDirectory { get; set; }

     
    }

``` 

I should have private setters, but meh...  Let's talk about some interesting stuff: the class inherits from `StartupContext`, defined in `StartItUp`, this is useful for passing values or using helpers like Autofac extension methods (see below). It also has a dependency on `IAppBuilder` because we are in an OWIN app and it will be needed to setup the web stuff.

Let's see a bit of logging configuration (with Serilog)

```csharp
 public class ConfigureLogging
    {
        public void Run(AppSettings cfg)
        {
            var tpl = "{Timestamp:G} |{Level}|{Message}{NewLine:l}{Exception:l}";

            var logConfig = new LoggerConfiguration();
            if (cfg.IsDebug)
            {
                logConfig.MinimumLevel.Debug();
            }
            else
            {
                logConfig.MinimumLevel.Information();
            }

            logConfig
                .WriteTo.Trace(outputTemplate: tpl,restrictedToMinimumLevel:LogEventLevel.Debug)
                .WriteTo.RollingFile(cfg.LogDirectory + "log-{Date}.log", outputTemplate: tpl, restrictedToMinimumLevel: LogEventLevel.Warning);
            LogManager.SetWriter(new SerilogAdapter(logConfig.CreateLogger()));
        }
    }

```

The interesting part here is that this class is automatically invoked by `StartItUp` before all other tasks. When the app will be ready to go in production, I'll add integration with a 3rd party logging service like LogEntries.

```csharp
 public class ConfigTask_0_RegisterServices
    {

        public void Run(AppSettings cfg)
        {
            cfg.ConfigureContainer(cb =>
            {
                cb.RegisterServices(asm: cfg.AppAssemblies);
                
                //AppSettings can be injected now 
                cb.Register(c => cfg).AsSelf().SingleInstance();
            });
        }
    }

```

We're configuring Autofac based on conventions: classes ending in "Service","Query" or "Manager" are automatically added by the `RegisterServices` method defined in `StartItUp.Autofac` and I can easily add other suffixes if I need to.

Let's set up the data mapper configuration, [SqlFu](https://github.com/sapiens/SqlFu) obviously.

```csharp
  public class ConfigTask_0_SqlFu
    {
        public void Run(AppSettings cfg)
        {
            
            SqlFuManager.Configure(c =>
            {
                c.AddProfile(new SqlServer2012Provider(SqlClientFactory.Instance.CreateConnection), cfg.DbConnection);
                c.AddNamingConvention(t => t.Name.EndsWith("Item"), t => new TableName(t.Name.SubstringUntil("Item")));
                c.RegisterConverter(o => o == null ? null : new UserId((Guid)o));
                c.RegisterConverter(o => o != DBNull.Value && ((int)o) > 0);
                
                c.OnCommand = cmd => this.LogDebug(cmd.FormatCommand());
            });

            
            cfg.ConfigureContainer(cb=> cb.RegisterInstance(SqlFuManager.GetDbFactory<MainDb>("default")).As<IMainDb>().SingleInstance());            

        }
    }

```

See how I configure the mapper and update the DI Container configuration. We keep related things in the same place.

Now, the (N)EventStore. 

```csharp

 public class ConfigTask_2_EventStore
    {

             
        public void Run(AppSettings cfg)
        {
            
            var es = Wireup.Init()
                .UsingSqlPersistence(cfg.Container().Resolve<IMainDb>())
                .WithDialect(new MsSqlDialect())
                .InitializeStorageEngine()
                .UsingJsonSerialization()
                .HookIntoPipelineUsing( new DispatchMessages(cfg.Container().Resolve<IDomainBus>().GetDispatcher()))
                .LogTo(t=>new LogAdapter(t))
                .Build();
            cfg.ConfigureContainer(cb =>
            {
                cb.RegisterInstance(es).As<IStoreEvents>().SingleInstance();
            });

        }
       
      

        class DispatchMessages:IPipelineHook
        {
            private readonly IDispatchMessages _bus;

            public DispatchMessages(IDispatchMessages bus)
            {
                _bus = bus;
            }

            public void Dispose()
            {
                
            }

            public ICommit Select(ICommit committed)
                => committed;

            public bool PreCommit(CommitAttempt attempt)
            =>true;

            public void PostCommit(ICommit committed)
            {
                var events = committed.Events.Select(d => d.Body).Cast<DomainEvent>().ToArray();
#if DEBUG
                this.LogDebug($"Publishing {events.Length} events: {events.Select(d=>d.GetType().ToString()).StringJoin()}");                
#endif
                _bus.Publish(events);
            }

            public void OnPurge(string bucketId)
            {
              
            }

            public void OnDeleteStream(string bucketId, string streamId)
            {
              
            }
        }
    }

```

As you can see, there is a bit of config here, including the definition of an event store hook that will be automatically invoked by NEventStore to publish the persisted events. Also we're using the DI Container but we update its configuration too. Once again, we keep relevant things in one place.  

My current project is a Nancy app, so besides other stuff I do need to bootstrap it and nancy has its own bootstrapper (ha!). So, I need to configure OWIN but also integrate with Nancy's  bootstrapper. How hard can it be?

```csharp
 public class ConfigTask_6_WebFramework
    {
        public void Run(AppSettings cfg)
        {
            var owin = cfg.ContextData["owin"] as IAppBuilder;
            owin.UseCookieAuthentication(new CookieAuthenticationOptions()
            {
                CookieName = "_dfauth",
                SlidingExpiration = true,
                ExpireTimeSpan = TimeSpan.FromDays(7)
            }, PipelineStage.Authenticate);
            
            
            owin.UseNancy(opt => opt.Bootstrapper = new NancyBoot(cfg));
            
            owin.UseStageMarker(PipelineStage.MapHandler);
        }
    }
    
 //nancy bootstrapper
  public class NancyBoot : AutofacNancyBootstrapper
    {
        private readonly AppSettings _cfg;

        public NancyBoot(AppSettings cfg)
        {
            _cfg = cfg;
        }

      /* removed other settings */

        protected override ILifetimeScope GetApplicationContainer() => _cfg.Container();
       
    }
    
```
When adding Nancy, we specify manually what bootstrapper to use and the bootstrapper will use our configuration object to return the Autofac instance when Nancy needs it. That wasn't that hard, was it?

 OK, now we have our tasks, we need one more thing:
 
 ```csharp
 
  public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            StartIt.Up(new AppSettings(app));
        }
    }
 
 ```

Yep, you need to invoke `StartItUp` manually, no automagic involved. This is required in order to pass things to AppSettings. And later, I'd have to add some code to  read the settings from web.config.

So, you've seen how we can organize the startup tasks in a maintainable manner. Very easy to add/change stuff, easy enough to integrate with other libraries and you can even unit test the tasks. Go on [Install-Package StartItUp](https://www.nuget.org/packages/StartItUp/2.0.0)!