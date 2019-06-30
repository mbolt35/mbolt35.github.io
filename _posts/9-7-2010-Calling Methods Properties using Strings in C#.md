---
layout: post
title: Calling Methods and Properties by String in C#
---
One of my favorite things about C# is the strict typing. Strict typing forces the user to follow good design principles (most of the time). However, it does prevent you from being able to use "true" dynamic objects (Note: there are ways that you can use generics and operator overloading to create a "dynamic" object, but they will not be discussed here). And while you can always use the [dynamic](http://msdn.microsoft.com/en-us/library/dd264736.aspx) keyword, you're still unable to call a method by string (like in AS3). When I say "calling a method/property by string," I am referencing the following syntax in AS3:  
```actionscript
someObject["myMethod"]();
// or... 
someObject["myProperty"];
```
For example, let's assume I want to replace all tokens, `${token}`, in a `String` with the matching property of an object.  
  
### In AS3
```actionscript
var person:Person = new Person("Matt Bolt", 27);
var str:String = "My name is ${name}. I am ${age} years old.";

trace(replaceTokens(str, person));

/**
 * @private
 * this method applies to regular expression to the string, then 
 * replaces the tokens with values from the data object provided.
 */
private function replaceTokens(str:String, data:Object):String {
    const GENERIC_TOKEN:RegExp = /\$\{([a-zA-Z0-9_]+)\}/g;

    return str.replace(GENERIC_TOKEN, function():String {
        return data[ arguments[1] ];
    });
}
```
This example would produce the following output:  
```
My name is Matt Bolt. I am 27 years old.
```
  
You can find a full example here: [StringReplaceExample.as](http://github.com/mbolt35/OpenSource/raw/master/blogspot/as3/com/mattbolt/examples/StringReplaceExample.as)  
  
In the AS3 code, we're simply performing a regular expression find/replace for any `${token}` (where token = a property name). The replace handler looks at `arguments[1]` which contains the "token" String. We then use that to resolve the property of the passed data object by using the square brackets:  
```actionscript
return data[ arguments[1] ];
```
  ### In C#
```csharp
Person person = new Person("Matt Bolt", 27);
string str = "My name is ${Name}. I am ${Age} years old.";

Debug.WriteLine( ReplaceTokens(str, person) );

/// <summary>
/// This method uses a generic token substitution and returns the 
/// resulting string.
/// </summary>
private string ReplaceTokens(string str, object obj) {
    Regex genericToken = new Regex("\\$\\{([a-zA-Z0-9_]+)\\}");
    
    return genericToken.Replace(str, (Match e) => {
        // FIXME: This will NOT compile!
        return obj[ e.Groups[1].Value ];
    });
}
```
As you can see, the C# version is **almost** identical to the AS3 version in terms of general syntax. However, note that `return obj[ e.Groups[1].Value ];` isn't going to allow you to compile. This brings us back to the initial problem of : **_How do we access a method/property on an object by using a string?_**  
  
This is done by using two classes: `Type` and `PropertyInfo` which exist in the `System.Reflection` package.  
```csharp
private T InvokeByString<T>( object obj, 
                             string methodName, 
                             params object[] parms ) 
{
    Type objType = obj.GetType();

    return (T)objType.InvokeMember(
        methodName, 
        BindingFlags.InvokeMethod | 
            BindingFlags.Public | 
            BindingFlags.Static, 
        null, 
        obj, 
        parms);
}

/// <summary>
/// This method invokes a property getter using a property name
/// </summary>
private T InvokePropertyByString<T>( object obj, 
                                     string propertyName ) 
{
    Type type = obj.GetType();
    PropertyInfo pInfo = type.GetProperty(propertyName, typeof(T));

    return (T)pInfo.GetValue(obj, null);
}
```
To make these methods more accessible, I added them to a static class here: [TypeUtil.cs](http://github.com/mbolt35/OpenSource/raw/master/silverlight/Bolt/CSharp/Util/TypeUtil.cs)  
  
So, to finalize our C# example, let's substitute the use of `TypeUtil` where our compile error was. Now, we have:  
```csharp
Person person = new Person("Matt Bolt", 27);
string str = "My name is ${Name}. I am ${Age} years old.";

Debug.WriteLine( ReplaceTokens(str, person) );

/// <summary>
/// This method uses a generic token substitution and returns the 
/// resulting string.
/// </summary>
private string ReplaceTokens(string str, object obj) {
    Regex genericToken = new Regex("\\$\\{([a-zA-Z0-9_]+)\\}");
    
    return genericToken.Replace(str, (Match e) => {
        return TypeUtil.InvokePropertyByString<string>(
            obj, 
            e.Groups[1].Value);
    });
}
```
  
The full example is located here: [StringReplaceExample.cs](http://github.com/mbolt35/OpenSource/raw/master/blogspot/silverlight/Bolt/CSharp/StringReplaceExample.cs)
