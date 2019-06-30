---
layout: post
title: After Two Years - AS3 Vector.<T> Review
---
Let me start off initially by saying that this post is pretty much a rant with examples concerning the Flash Platform's `Vector.<T>` class and the mindless semantic differences between it and the `Array` class. I suppose this is a fairly out of date topic, being as the `Vector.<T>` class appeared in the Flash Player 10.0 release. However, I have not noticed any changes or improvements to the APIs since the 10.0 release, and thus the complaining commences:  
  
_Just a note, that I use the "shorthand" instantiation of both Array and Vector. If you're unfamilar with this syntax you can read more [here](http://help.adobe.com/en_US/as3/dev/WS5b3ccc516d4fbf351e63e3d118a9b90204-7ee5.html#WSB1F41227-C612-4f33-A00E-CE84C1913E1C), or here's a quick rundown:_  
```actionscript
// These produce the same Array *structure*, they are not
// strictly equivalent (===)
var oneWay:Array = new Array(1, 2, 3);
var anotherWay:Array = [ 1, 2, 3 ];
 
// Each of these result with the same Vector.<int> 
// *structure*, however they are not strictly 
// equivalent (===).
var v1:Vector.<int> = new Vector.<int>(3);
v1[0] = 1; v1[1] = 2; v1[2] = 3;
var v2:Vector.<int> = new <int>[ 1, 2, 3 ];
var v3:Vector.<int> = Vector.<int>([ 1, 2, 3 ]);
```
  
## Conversion

#### Array

The conversion from Array to Vector.<T> is done via a top-level conversion method, similar to XML(xml:String). This is a good choice in my opinion. This is done like so:  
 ```actionscript
var arr:Array = [ 1, 2, 3 ];
var vec:Vector.<int> = Vector.<int>(arr);
trace(vec);
// Outputs: 1,2,3
```
  

#### Vector

The conversion from Vector to Array doesn't exist as a utility in AS3. Um, ok. So, I guess we write our own. Since we want to be able to convert any _T-type_ Vector, let's write a static helper function that accepts a \* parameter:  
```actionscript
public static function vectorToArray(vector:*):Array {
    var result:Array = [];
    for (var i:int = 0; i < vector.length; ++i) {
        result[i] = vector[i];
    }
    return result;
}
```  
  
There are problems with this, most notably:
*   No typing on the vector object, so it's going to execute substantially slower.
*   There's no type checking at compile time, so we could pass anything to this method and end up with unexpected run-time errors.

  
_**No typing on the vector object**_  
The first bullet point could be solved if we get rid of the generic helper method, and just in-lined the conversion via an anonymous function or loop.  
```actionscript
var vec:Vector. = new <int>[ 1, 2, 3 ];
var arr:Array = function(v:Vector.<int>):Array {
    var result:Array = [];
    for (var i:int = 0; i < v.length; ++i) {
        result[i] = v[i];
    }
    return result;
}(vec);
```
  
I personally like the anonymous function, just to make it clear that we're executing a conversion function on the Vector input and getting an Array output, but it might seem too "cute" for co-workers, so you may find work a more friendly place if you just use a for loop.  
  
_**Unexpected run-time errors**_  
The second bullet point, we can "solve" in a few ways as well. Let's say that my application has decent error handling, which requires a semi-intelligent use of Error objects when expected exceptions occur. Since the point is to avoid unexpected run-time errors, let's validate our input, and throw a more reasonable error, which can be caught and handled:  
```actionscript
// This constant is set to the fully qualified class name for Vector
private static const VECTOR_CLASS:String = "__AS3__.vec::Vector";
 
// Use flash.utils.describeType() on the input, and verify type 
// of the object is a Vector class. This is probably fairly
// expensive, a valid point to my argument.
public static function isVector(vector:*):Boolean {     
    var type:String = flash.utils.describeType(vector).@name;
    return type.indexOf(VECTOR_CLASS) != -1;
}
```
  
Using these new helpers, let's validate our original helper method before performing the iteration:  
```actionscript
public static function vectorToArray(vector:*):Array {
    if (!isVector(vector)) {
        throw new ArgumentError("Illegal argument, non-Vector for use with vectorToArray().");
    }
     
    var result:Array = [];
    for (var i:int = 0; i < vector.length; ++i) {
        result[i] = vector[i];
    }
    return result;
}
```
### Thoughts

Why does it have to be so complex? Why is it so simple to convert an Array to a Vector, but not the other way around? The reason I have a problem with the lack of conversion is due to the nature of `Vector` and `Array`. Vectors are not only nice type-friendly collection objects, but they're also are very fast during iteration. They're also unlike `Array` in that they're strictly indexed (in order). Because of this, certain operations like element removal via `splice()` and `shift()` are slower using a `Vector` compared to with an `Array`. This makes sense because the `Vector` has the additional overhead of maintaining index order internally.  
  
So, if I'm executing some sort of filtering logic on a collection of elements, it's typically best for me to stick with `Array`. However, in terms of creating a user-friendly API, I prefer strongly typed `Vector` structures.  
  
What I am left with is either the decision to just stick with `Array` and avoid `Vector`, or implement two way conversions (which we've already seen is a bit of a headache). Perhaps there are better ways to go about this. Let's continue.  
  

## Concatenation

### Array

Array + Array concatenation is fairly simple, and probably what anyone would expect:  
```actionscript
var arr:Array = [ 1, 2, 3 ]; 
var arr2:Array = [ 3, 2, 1 ]; 
var result:Array = arr.concat(arr2); 
trace(result); 
// Outputs: 1,2,3,3,2,1
```
  
The concat() method will also accept multiple Array objects:  
```actionscript
// ... 
var arr3:Array = [ 9, 8, 7 ]; 
result = arr.concat(arr2, arr3);  
trace(result); 
// Outputs: 1,2,3,3,2,1,9,8,7
```
  

### Vector

Vector + Vector concatenation is also fairly straightforward, and works similarly to Array:  
```actionscript
var vec:Vector.<int> = new <int>[ 1, 2, 3 ]; 
var vec2:Vector.<int> = new <int>[ 3, 2, 1 ]; 
var result:Vector.<int> = vec.concat(vec2); 
trace(result); 
// Outputs: 1,2,3,3,2,1
```
  
And just like Array's concat(), we can specify multiple Vector.<T> objects to concatenate:  
```actionscript
var vec3:Vector.<int> = new <int>[ 9, 8, 7 ]; 
result = vec.concat(vec2, vec3); 
trace(result); 
// Outputs: 1,2,3,3,2,1,9,8,7
```
  
Pretty much the same, right? Let's go back to Array.  
  

### Array

Array's concat() also let's you concatenate individual elements:  
```actionscript
var arr:Array = [ 1, 2, 3 ]; 
var result:Array = arr.concat(3, 2, 1); 
trace(result); 
// Outputs: 1,2,3,3,2,1
```
You can also use a combination of individual elements and Array objects:  
```actionscript
var arr:Array = [ 1, 2, 3 ]; 
var arr2:Array = [ 3, 2, 1 ]; 
var result:Array = arr.concat(arr2, 4, 5, 6); 
trace(result); 
// Outputs: 1,2,3,3,2,1,4,5,6
```
  

### Vector

For some reason, whoever wrote the Vector `concat()` method decided he/she would subtly rip-off the entire AS3 community by silently killing the ability to pass individual elements to the `concat()` method.  
```actionscript
var vec:Vector.<int> = new <int>[ 1, 2, 3 ];
var result:Vector.<int> = vec.concat(3, 2, 1);
trace(result);
// Unfortunately, this code ERRORS with: "Type Coercion failed" 
// Cannot convert 3 to __AS3__.vec.Vector.<int>."
```
  
  
So, clearly you are only allowed to pass Vector objects with the same _T-type_ to the `concat()` method. This seems wasteful:  
```actionscript
/** 
 * So, if I want to create a new Vector.<int> that 
 * appends 3, 2, 1 without modifying the original, 
 * I need to create *another* new Vector.<int>, just 
 * to pass the values to concat(). That's one extra
 * Vector.<int> instantiation. Three total Vector.<int> 
 * objects, while only 2 should be needed:
 *   -- One, the original vec:Vector.<int>.
 *   -- Two, the new <int>[ 3, 2, 1 ] to pass the concat() 
 *      parameter.
 *   -- Three, the result:Vector.<int>
 */
var vec:Vector.<int> = new <int>[ 1, 2, 3 ];
var result:Vector.<int> = vec.concat( new <int>[ 3, 2, 1 ] );
trace(result);
 
/** 
 * Or, we can save the trouble of creating an extra 
 * Vector.<int> by shallow copy and appending the 
 * 3, 2, 1 manually:
 */
var vec:Vector.<int> = new <int>[ 1, 2, 3 ];
 
// Shallow Vector.<int> copy
var result:Vector.<int> = vec.concat(); 
result.push(3, 2, 1);
trace(result);
// Outputs: 1,2,3,3,2,1
```
  
We have to write three lines of code to create, copy, and append for `Vector`, while this can be done with 2 lines (preserving the original) with `Array`.  
  

### Thoughts

Again, we have a very similar set of APIs on two collection classes, and while the semantics of both seem identical (even after reading the documentation), they have subtle, yet annoying, differences.  
  

## Sorting

### Array

I'm certainly a fan of the native array sorting that can be done in flash. It's very easy to use, and much faster than a homemade AS3 sorting function. Let's talk about the two sorting methods available on Array:  
  
##### Custom Sorting Function
The `sort()` method allows you to pass a few different types of parameters, which you can read about in the [AS3 Array sort() documentation](http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/Array.html#sort()), but for the sake of time, let's use the custom sort function method:  
```actionscript
// We return -1 if val1 is first, 1 if val2 is first, and 0
// if we don't care - just return a random value in the range
// (-1, 1) inclusive.
function shuffle(val1:int, val2:int):int {
    var random:int = Math.floor(Math.random() * 3) - 1;
    return random;
}
 
var toSort:Array = [ 1, 2, 3, 4, 5 ];
toSort.sort(shuffle);
trace(toSort);
```
  
### Vector

Not surprisingly, the Vector class has a `sort()` method as well, and would you believe that it takes the exact same parameters as the `Array` sort. At least, from what I can read in the ASDocs and from all my tests, they take the same type of arguments.  
```actionscript
var vec:Vector.<int> = new <int>[ 1, 2, 3, 4, 5 ];
vec.sort(shuffle);
trace(vec);
```
  
Alas, it seems the same lazy asshole who wrote the Vector `concat()` must have also copy and pasted the `sort()` method from `Array` because according to the Vector ASDocs, here's how I should use the sort options with the `sort()` method.  
```actionscript
var vec:Vector.<int> = new <int>[ 1, 2, 3, 4, 5 ];
vec.sort(Array.DESCENDING);
trace(vec);
// Outputs: 5,4,3,2,1
```

Of course! Now it's ok to pass Array data to a Vector method!  
  

### Array

Being able to use a custom sort() function on Array a huge plus since the there may not always be a clear way to sort a collection of elements. Even though the guts of the sort() method run natively, if you use a custom sorting function, it has to be called on N elements, and obviously, the code execution isn't native. Either way, sort(compare) is still fast, and to make it possible to do a semi-custom sort, the Array APIs added an elegant and simple hook for sorting objects using their properties: `sortOn()`:  
```actionscript
package { 
    public class MyObject {
        public var alpha:String = "";
        public var val:int = 0;
         
        public function MyObject(alpha:String, val:int) {
            this.alpha = alpha;
            this.val = val;
        }
         
        public function toString():String {
            return "[alpha: " + this.alpha + ", val: " + this.val + "]";
        }
    }
}
 
// test array of objects
var arr:Array = [
    new MyObject("A", 10),
    new MyObject("Z", 2),
    new MyObject("D", 16),
    new MyObject("D", 37),
    new MyObject("D", 5),
    new MyObject("t", 3000),
    new MyObject("R", 732)
];
 
/* Sorting on a single integer field */
/* 
 [alpha: Z, val: 2],
 [alpha: D, val: 5],
 [alpha: A, val: 10],
 [alpha: D, val: 16],
 [alpha: D, val: 37],
 [alpha: R, val: 732],
 [alpha: t, val: 3000]
*/
arr.sortOn("val", Array.NUMERIC);
 
 
/* Sorting on single string field */
/*
 [alpha: A, val: 10],
 [alpha: D, val: 5],
 [alpha: D, val: 16],
 [alpha: D, val: 37],
 [alpha: R, val: 732],
 [alpha: Z, val: 2],
 [alpha: t, val: 3000]
*/
arr.sortOn("alpha");
 
 
/* Sorting on two fields, using different options for each field */
/*
 [alpha: A, val: 10],
 [alpha: D, val: 5],
 [alpha: D, val: 16],
 [alpha: D, val: 37],
 [alpha: R, val: 732],
 [alpha: t, val: 3000],
 [alpha: Z, val: 2]
*/
arr.sortOn([ "alpha", "val" ], [ Array.CASEINSENSITIVE, Array.NUMERIC ]);
```
From an AS3 perspective, this is such a powerful tool. Native speed, custom property hooks, and basic sorting strategies all implemented in one line of code that's immediately easy to understand.  
  
### Vector

Vector does not have the method: `sortOn()`. This forces us to use the sort() method, or manually swap things in AS3. I just don't understand!  
  

## Final Thoughts

I've been using the Vector class since it's addition to the flash APIs for the Flash Player 10 release, and I've definitely seen how much speed improvement you can get by simply just swapping in Vector for Array. Let's also not forgot how great it is to finally be able to associate a single type with a collection of elements. Let me also commend the Flash Platform team for taking a non-existing generics standard, and cutting a few corners to bring developers something that "was as close as possible" at the time.  
  
However, in the past few years, the only real reason (unfortunately) I have used `Vector` is for type clarity, and because it didn't really matter whether I used `Array` or `Vector`. To me, `Vector.<int>` is a lot less confusing to look at than just plain ole `Array`. In most of the flash games we've written at Electrotank, we just don't use a lot of super heavy iteration like you'd see in the "300,000 pixel pushing particles demo," and we quickly noticed that swapping in `Vector` for every occurrence of Array mindlessly was **NOT** the way to go. In fact, the conversion problem I mentioned earlier will turn your code into spaghetti very quickly, and many times, we saw a slower performance (especially using `splice()` and `shift()`).  
  
Of course, I don't cover all of the differences between `Array` and `Vector` in this article, as I've chosen to focus on the two that effect me the most. In my opinion, using a faster, strongly typed data structure shouldn't be a burden on the developer. In fact, I may go as far as to say that you should be able to pass a `Vector` anywhere you can pass an `Array`. At the minimum, a native conversion from Vector -> Array is needed.  
  
My disappointment is in the lack of effort that has been applied to the `Vector` class in the past few years. The subtle differences between `Vector` and `Array` make them far too difficult to use together. Collections are supposed to be available to a developer to use for different circumstances, and in most cases (like Java for example), collections can be easily converted into another. As a product, you end up with well organized, high performing code.  
  
Unfortunately, the Flash Platform didn't see it the same way.  
  
_The claims I've made in this post are based on my general experience with AS3 and the Flash Player 10 and higher. I'm a bit over the top at times, some of the views expressed may be over exaggerated, so take it all with a grain of salt. It'd be unfair for me to dish out comments like this if I wasn't prepared to accept corrections or disagreement, so please feel free to set me straight, or disagree!_
