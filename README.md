# Warden Performance Watcher

![Warden](http://spetz.github.io/img/warden_logo.png)

**OPEN SOURCE & CROSS-PLATFORM TOOL FOR SIMPLIFIED MONITORING**

**[getwarden.net](http://getwarden.net)**

|Branch             |Build status                                                  
|-------------------|-----------------------------------------------------
|master             |[![master branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Performance.svg?branch=master)](https://travis-ci.org/warden-stack/Warden.Watchers.Performance)
|develop            |[![develop branch build status](https://api.travis-ci.org/warden-stack/Warden.Watchers.Performance.svg?branch=develop)](https://travis-ci.org/warden-stack/Warden.Watchers.Performance/branches)

**PerformanceWatcher** can be used for CPU and RAM monitoring. Accepts optional parameter *machineName* to work with the remote host.

### Installation:

Available as a **[NuGet package](https://www.nuget.org/packages/Warden.Watchers.Performance)**. 
```
dotnet add package Warden.Watchers.Performance -v 2.0.0-beta-1
```

### Configuration:

 - **EnsureThat()** - predicate containing a received *ResourceUsage* that has to be met in order to return a valid result.
 - **EnsureThatAsync()** - async predicate containing a received *ResourceUsage* that has to be met in order to return a valid result.
 - **WithPerformanceProvider()** - provide a  custom *IPerformance* which is responsible for calculating the resource usage. 

**PerformanceWatcher** can be configured by using the **PerformanceWatcherConfiguration** class or via the lambda expression passed to a specialized constructor.

Example of configuring the watcher via provided configuration class:
```csharp
var configuration = PerformanceWatcherConfiguration
    .Create(delay: TimeSpan.FromMilliseconds(200))
    .EnsureThat(usage => usage.Cpu < 30 && usage.Ram < 3000)
    .Build();
var performanceWatcher = PerformanceWatcher.Create("Performance watcher", configuration);

var wardenConfiguration = WardenConfiguration
    .Create()
    .AddWatcher(performanceWatcher)
    //Configure other watchers, hooks etc.
```

Example of adding the watcher directly to the **Warden** via one of the extension methods:
```csharp
var wardenConfiguration = WardenConfiguration
    .Create()
    .AddPerformanceWatcher(cfg =>
    {
        cfg.EnsureThat(usage => usage.Cpu < 30 && usage.Ram < 3000);
    }, delay: TimeSpan.FromMilliseconds(200))
    //Configure other watchers, hooks etc.
```

Please note that you may either use the lambda expression for configuring the watcher or pass the configuration instance directly. You may also configure the **hooks** by using another lambda expression available in the extension methods.

### Check result type:
**PerformanceWatcher** provides a custom **PerformanceWatcherCheckResult** type which contains additional values.

```csharp
public class PerformanceWatcherCheckResult : WatcherCheckResult
{
    public TimeSpan Delay { get; }
    public ResourceUsage ResourceUsage { get; }
}
```
### Custom interfaces:
```csharp
public interface IPerformance
{
    Task<ResourceUsage> GetResourceUsageAsync();
}

public class ResourceUsage
{
    public double Cpu { get; }
    public double Ram { get; }
}
```

**IPerformance** is responsible for calculating the resource usage. It can be configured via the *WithPerformanceProvider()* method. By default it is based on the **[Performance Counter](https://msdn.microsoft.com/en-us/library/system.diagnostics.performancecounter(v=vs.100).aspx)**.