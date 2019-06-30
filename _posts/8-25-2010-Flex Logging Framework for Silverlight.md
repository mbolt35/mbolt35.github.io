---
layout: post
title: Flex Logging Framework for Silverlight
---
I've taken some time and created a logging framework in C# that is very similar (in fact, uses close to identical logic in some cases) to that of flex. It's super flexible and easy to use.  
  
You'll find the source at: [http://github.com/mbolt35/OpenSource/tree/master/silverlight](http://github.com/mbolt35/OpenSource/tree/master/silverlight)  
  
The `Bolt.AS3.Logging` package contains the necessary classes/interfaces and logging targets. I've also created a very basic example here:  [LoggingExample.cs](http://github.com/mbolt35/OpenSource/raw/master/blogspot/silverlight/Bolt/CSharp/LoggingExample.cs)  
  
Which generates the following output:  
```
[INFO] Bolt.CSharp.LoggingExample This is a Logging Test  
[DEBUG] Bolt.CSharp.LoggingExample You can log like this: Test, 1, 2, 3  
[WARN] Bolt.CSharp.LoggingExample But it's preferable to use this: Test, 1, 2, 3  
[ERROR] Bolt.CSharp.LoggingExample You can have multiple parameters in your log: True - that are any type (object) - Test, 1, 2, 3,  
and multiline is supported: 25  
[FATAL] Bolt.CSharp.LoggingExample The End...  
```
  
One thing that I find extremely useful about the logging is the way in which a logging instance is defined:  
```csharp
/// <summary>
/// Creates a read-only static logger instance with the 
/// fully-qualified class name as the category.
/// </summary>
private static readonly ILogger log = Log.GetLogger(
    MethodBase.GetCurrentMethod().DeclaringType.FullName);
```
  
Note that the `MethodBase` is in the `System.Reflections` package.  
  
This line of code is great because it can be templated and added to any new class you create, and without ever changing the code, the log category will be set to the fully-qualified class name of the object in which it's created!  
  
Hope you all enjoy this code, and find it useful!
