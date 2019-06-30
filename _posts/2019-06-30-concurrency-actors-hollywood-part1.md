---
layout: post
title: "Concurrency, Actors, and Hollywood: Part 1"
---
A computational model once thought to be only implementable by the notable Erlang neckbeard wizards, "Actors" have become popular discussion point when it comes to concurrency and modern day computer languages. In this series of posts, we'll cover the problem areas of writing concurrent code in classic object oriented languages like Java and C#, and take a look at what has been done to "improve" concurrency as a language component. Finally, we'll focus in on "Actors" and go over what they bring to the concurrency discussion. However, before we get there, let's start with the "old school" object oriented approach to concurrency and shared memory access to understand the importance of this topic in the first place.

### Object Oriented Concurrent Utilities
Writing highly concurrent code could involve layers of asynchronous operations that require _efficient_ shared memory access, all of which the developer has to orchestrate. How?

Languages like Java and C# offer concurrency primitives and collections for writing thread-safe code, and while one could argue they do provide a nice utility, managing shared memory access across multiple threads using these utilities can be awkward and bug prone. However,  in my opinion, the worst problem is how they mimic common OO utility for the sake of consistent API. 

### A "Simple" Example
Take the following snippet of code for example:
```csharp
// System.Collections.Generic.Dictionary<K, V>
private Dictionary<string, int> _map = new Dictionary<string, int>();

// Set the value in our map using the provided key. If the key already exists, do nothing.
public void SetValue(string key, int value) 
{
    // Check to see if key exists. If so, do nothing
    if (_map.ContainsKey(key))
    {
        return;
    }

    // Set the Value
    _map[key] = value;
}
```
Assuming that this code is part of larger application that utilizes multiple threads, you will run into issues if multiple threads set values for the same key at the same time. 

But this is expected right? This example doesn't use any of the concurrent utilities we mentioned earlier, so of course multiple threads are going to cause issues. There is the concurrent utility: `ConcurrentDictionary<K, V>`. 

> All we should have to do is replace `Dictionary` with `ConcurrentDictionary`, and our bug disappears right? 

Unfortunately, it doesn't. In fact, for the exact same reason, it's still possible for multiple threads to overwrite values for the same key. In order for the `SetValue` method to be truly thread safe, the `ContainsKey` and set need exclusive access to memory. This should do the trick:
```csharp
// System.Collections.Concurrent.ConcurrentDictionary<K, V>
private ConcurrentDictionary<string, int> _map = new ConcurrentDictionary<string, int>();

public void SetValue(string key, int value) 
{
	lock(_map) {
	    // Check to see if key exists. If so, do nothing
	    if (_map.ContainsKey(key))
	    {
	        return;
	    }

	    // Set the Value
	    _map[key] = value;
	}
}
```
> But couldn't we just use a regular `Dictionary<K, V>` here if we're going to lock? 

Yes, we could, and that would actually be more "efficient" since `Dictionary` has less overhead than `ConcurrentDictionary`. To go even further, instead of using a `lock` around the entire check and set, you could use a [`ReaderWriterLockSlim`](https://docs.microsoft.com/en-us/dotnet/api/system.threading.readerwriterlockslim?view=netcore-2.2)  to more efficiently manage locking around read and write access separately. 

> That's great and all, but you've lost me. What's the point of `ConcurrentDictionary<K, V>`? 

The internals of `ConcurrentDictionary<K, V>` are actually different than the two previous solutions, where [`TryAddInternal`](https://github.com/microsoft/referencesource/blob/e0bf122d0e52a42688b92bb4be2cfd66ca3c2f07/mscorlib/system/collections/Concurrent/ConcurrentDictionary.cs#L808-L953) is locking per hash bucket. Instead of using the `Dictionary` methods with `ConcurrentDictionary`, there are specific methods designed to execute atomically. If you're using a `ConcurrentDictionary<K, V>`, the correct way to write this code is:
```csharp
// System.Collections.Concurrent.ConcurrentDictionary<K, V>
private ConcurrentDictionary<string, int> _map = new ConcurrentDictionary<string, int>();

public void SetValue(string key, int value) 
{
	var didSetValue = _map.TryAdd(key, value);
}
```
This will atomically set the `value` for a specific `key` and return `true` if the `key` was set, and `false` if the key already existed. 

Phew! This is _a lot_ of information, and the problem we're trying to solve isn't that complicated. The important things to note:
* Concurrency Utilities are not always drop in solutions.
* There are a lot of ways to fence memory access, many of which are inefficient.
	* **NOTE:** Inefficiency may not a major problem in certain cases (not much thread contention), but for most of the exercises in these posts, we'll assume we need to maximize efficiency.
* There can exist **many** concurrency primitives, collections, and utilities designed to _assist_ the developer, but they can take years to master and this knowledge doesn't always transfer across platforms and languages. 

Modern day languages  are moving towards "getting off the ground" quickly with emphasis on fast iteration. Spending countless hours trying to build an application relying on awkward concurrency design patterns does not meet this criteria, and as such, the object oriented concurrency primitives are slowly becoming a thing of the past. So, given all of the pitfalls we're aware of, what approach could be taken to make things better?

## Promises for Future Tasks
In the last section, we covered a use-case that involved multiple `Threads`, a vessel that can manage a unit of work asynchronously. They do not provide any APIs for flow control, so any execution ordering or throttling of a Thread has to be written by the developer. What if we could abstract away the Thread flow control boilerplate into a unit of work that could execute synchronously or asynchronously? Turns out, we can.

One of the more prominent shifts in concurrent programming idioms is the notion of a `Promise`. Different languages introduced different terminology, for instance:
* C# uses `Task`
* Java uses `Future`
* Javascript uses `Promise`

While the APIs and syntax may be different, the core principle is the same: An object that acts as a proxy for a unit of work. In other words, an object which represents specific work that can be queried for a result.

> How does this have anything to do with concurrency?

Promises allow a developer to string together synchronous code with _potentially_ asynchronous code as if all code was synchronous. Promises abstract away the boilerplate for managing asynchronous code. Where most developers are very used to:
```javascript
a();
b();
c();
```
The ordering of code represents the order in which it executes. For a synchronous application, these functions execute serially, the next starts after the previous completes. However, If any of these methods execute asynchronously, we could see unexpected results:
```javascript
a(); // a starts
// a completes
b(); // b starts
c(); // c starts
// c completes
// b completes
```
This is a problem if `c()` depends on results from `b()` because `c()` starts executing well before `b()` completes. With promises, these problems are easily organized in the familiar serial code structure:
```javascript
a()
  .then(b)
  .then(c);
```
In short, this API gives the developer a method of flow control for writing asynchronous code. It doesn't matter if `b()` runs in another thread because I know that once it completes, `c()` will run.

> That sounds great and all, but it seems like the developer should also still be aware of which promises are asynchronous in order to maximize efficiency, or the number of concurrent processes allowed to run at any given moment.

Absolutely. Just like any programming idiom or pattern, it's important to understand context. Promises don't try to automatically solve the concurrency dilemma's for you, but they do allow you to better control the flow of data through your application, which lets you focus more on writing code "naturally" instead of constantly fencing memory access, littering your code with  `locks`, and coming up with mildly insane solutions for race conditions.

> Even with this enhanced flow control for asynchronous code, don't you still have to worry about shared memory access?

Unfortunately, you do still have to implement concurrency primitives where you are sharing memory. So, it does seem like we've made an improvement, but we're still far away from being able to completely get rid of locking and other concurrency primitives.

## Next Week, Part 2
We'll dive deeper into the relationship in the terminology `asynchronous` and `concurrent`. We'll also discuss how an intelligent `Scheduler` can organize `asynchronous` work to maximize `concurrency`, and how we can leverage flow control with a scheduler to create more efficient concurrent code and which platforms have taken advantage of this.

This should hopefully bring us to `Actors` and how they fit into the big picture. 