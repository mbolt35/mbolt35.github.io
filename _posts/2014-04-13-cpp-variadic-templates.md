---
layout: post
title: C++11 Variadic Templates
---

In recent news, I've been revisiting my programming roots and tinkering with some of the latest C++11 has to offer. There are many new **awesome** features, including [variadic templates](http://en.wikipedia.org/wiki/Variadic_template). These templates give you the ability to define an arbitrary list of types in your template parameters. This removes the need to define N template parameters using specialization. Consider the following naive example tcontainer:
```cpp
template<class T1, class T2, class T3, class T4>
struct tcontainer {
  tcontainer(T1 a, T2 b, T3 c, T4 d) 
    : a(a), b(b), c(c), d(d) { }

  T1 a;
  T2 b;
  T3 c;
  T4 d;
};
```
Besides being horrific to look at and a pain to maintain, it limits the implementer to, in our case 4, types. This type of implementation always reminds me the first time I realized that the `Action<T>` was implemented with **16** [variations](http://msdn.microsoft.com/en-us/library/dd402872%28v=vs.110%29.aspx). _Yikes._

### Varia-wat?

Now there is a syntax that allows us to compress the N types down to a single _"Packed Parameter"_ using the _splat_ (...) syntax. Here is a very basic example:
```cpp
#include <iostream>

// print out a single T argument
template<class T>
void print(T value) {
    std::cout << value << std::endl;
}

// print out T argument, then expand Ts argument until there is 
// one remaining. the single argument will then go to the above
// print
template<class T, class... Ts>
void print(T value, Ts... values) {
    std::cout << value << std::endl;
    print(values...);
}
```
Note that we are essentially dequeing the first element (`T`) and _recursively_ (kind of) passing the rest of the arguments until there is only one argument left. The other wonderful thing about templates is if any `T` passed to the print method doesn't have an overloaded stream operator `<<`, then the error is compile-time. If we were to call this method, here's a very simplified view of how it might expand:
```cpp
std::string s = "hello";
int32_t i = 35;
double d = 203.451;
char c = 'w';

// print method
print(s, i, d, c);

/* Expansion */
// template<std::string, ...>
print(std::string value, int32_t ivalue, double dvalue, char cvalue);

// template<int32_t, ...>
print(int32_t value, double dvalue, char cvalue);

// template<double, ...>
print(double value, char cvalue);

// template<char>
print(char value); // this is template<class T> implementation so it stops here
```
A moment ago, I mentioned that the call to `print<T,Ts...>` was _kind of_ recursive. While it certainly has recursive appearance and qualities, the compiler is going to treat each variation as its own method. _The most important aspect of meta-programming to remember is that templates are resolved at compile time._ Personally, I'm constantly having to remind myself that template resolution must be known ahead of time.

### _Container (Tuple)_

Let's take a look at another example, this time using a class/struct. For all intents and purposes, you could really call this implementation a basic tuple. The `std::tuple` is a much more complete design, but it can be quite intimidating to look at for learning purposes. So, let's create a basic container class which will store a list of `Ts...` instances. Just like in our previous example, we'll need to create two basic implementations:

1.  **Link Implementation**: The purpose of this implementation is to extract the front T node, and pass the remaining Ts... onto the next link. 
	```cpp
	// container object must contain at least 1 T object. 
	template<class T, class... Ts>
	struct container : public container<Ts...> {

	    // create a typedef to represent the T type representing
	    // the value's type at this container level.
	    typedef T type;

	    // create a typedef to represent the container type in 
	    // which this container subclasses.
	    typedef container<Ts...> base_container;

	    // constructor accepts initialization values. store the T
	    // instance as this->value and pass expanded Ts... values 
	    // to our base container
	    container(T value, Ts... values)
	        : base_container(values...),
	          value(value)
	    { }

	    // value at this level of the container
	    T value;
	};
	```

5.  **Terminator Implementation**: The purpose of this implementation is to halt the "recursion." In other words, we want to continue to sub-class our container<class T, class... Ts> implementation until we have one remaining T instance. This implementation specializes the container to do just that. 
	```cpp
	// the terminating container class is a specialization of the link implementation.
    // whenever Ts... is empty (just a remaining T value), this class is used instead
    // of the link. 
    template<class T>
    struct container<T> {
  
      // just like with the link implementation, define a type for T
      typedef T type;
  
      // this type definition is a bit different than the link, and will be 
      // explained later -- basically, since we're terminating here, we'll
      // also want o set this implementations type as the "last."
      typedef container<T> base_container;
  
      // since there are no longer base classes, just store the value
      container(T value) : value(value) {
    
      }
  
      // value at this level of the container
      T value;
    };
	```

Now, let's create a factory method so we can let type inference do it's thing.
```cpp
// creates a new container instance using the values provided
template<class T, class...Ts>
container<T, Ts...> make_container(T value, Ts... values) {
  return container<T, Ts...>(value, values...);
}

// entry point
int main(int argc, const char* argv[]) {
  // create container<std::string, int, double, float>
  auto c = make_container(std::string("hello"), 32, 46.2, 0.6f);

  return 0;
}
```

### Access `value` Fields


Now that we can create an instance of our container, you'll immediately notice that if you print out the `value` field, it will output the `std::string` instance in the first element of our container creation `("hello")`. Since our `container<T, Ts...>` class uses the same field name (`value`) for each sub-class, then in order to extract a specific value of type `T`, we'll need pick the subclass that matches our desired value, cast our container, and _then_ access the `value` field. This is somewhat difficult to explain accurately, so here's an example to clarify:
```cpp
// our values
std::string s = "hello";
int i = 32;
double d = 46.2;
float f = 0.6f;

// refraining from using the "auto" keyword for clarity
container<std::string, int, double, float> c = make_container(s, i, d, f);

// this container instance hierarchy looks like this:
// container<std::string, int, double, float> 
//  : container<int, double, float>
//  : container<double, float>
//  : container<float>

// recall that the "T" for each subclass is the first type in the template 
// parameters. since c is of the type container<std::string, int, double, float>, 
// the value type is std::string
std::string s_value = c.value; 

// if I wanted to extract the int value, I would first need 
// to down-cast before accessing value
int i_value = static_cast<container<int, double, float>&>(c).value;

// for double...
double d_value = static_cast<container<double, float>&>(c).value;

// and finally, for float...
float f_value = static_cast<container<float>&>(c).value;
```

### Indexed Approach

Based on the above code to extract the desired `value` from our container, it seems that an implementer would have to know a lot about the implementation details of the container in order to actually use it. Let's provide an additional utility that will allow us to specify an index representing which type we'd like to access. We'll need to create two more template classes such that we can dive into the subclasses, and reduce the index until we hit `0`. Just like our _terminator_ class for container, we'll also need a specialized template for the `0-case`.
```cpp
// this class will traverse the container subclasses and subtract from the index until
// index == 0, in which case, our specialized implementation below represents the container
// subclass we were looking for
template<int index, class C>
struct container_index {

  // points to the "next" container type 
  typedef typename container_index<
    index-1,
    typename C::base_container
  >::container_type container_type;

  // points to the next T-type 
  typedef typename container_index<
    index-1,
    typename C::base_container
  >::type type;
};

// once our index has reached 0, we simply typedef the current container
// type as C, and the T-type as C::type
template<class C>
struct container_index<0, C> {

  // point to C instead of C::base_container
  typedef C container_type;

  // point to C::type instead of C::base_container::type
  typedef typename C::type type;
};
```
Here again, we're using the same pattern as we did in the first `print` example and with our `container<T, Ts...>` class, which is the use of _"recursive"_ access. The termination with this case is dependent on the value of `int index`, which is decreased by `1` each depth. The idea is that once the index is `0`, then the typedefs will point to the correct container type and `T`\-type.
```cpp
// consider the previous example's container type
typedef container<std::string, int, double, float> example_container;

// use container_index with index=2 to get the second subclass: container<double, float>
typedef typename container_index<2, example_container>::container_type i2_container;

// using the same approach, we can also get the T-type
typedef typename container_index<2, example_container>::type i2_type;

// create our container (typedef to example_container)
example_container c = make_container(std::string("hello"), 32, 46.3, 0.6f);

// if we know that the 2nd index yields a double value, we can use...
double i2_value = static_cast<i2_container&>(c).value;

/* or... */

// if we want to be a bit more generic, we can type i2_value as i2_type...
i2_type i2_value = static_cast<i2_container&>(c).value;

/* or... */

// we can simply use the "auto" keyword -- the choice is really based on context
auto i2_value = static_cast<i2_container&>(c).value;
```
The last step into turning this into a generic utility for use with our `container<T, Ts...>` class is to create a `get` method that takes `int index, class C` template parameters, define the return type as `typename container_index::type`, and then perform the `container_type` lookup and `static_cast` in the method body and return the `value`. This will look something like this:
```cpp
// create a "get" method that will extract the correct value using an index
// note that we use the container_index::type for the return type and 
// container_index::container_type as the type to cast to
template<int index, class C>
typename container_index<index, C>::type get(C& p_container) {
  typedef typename container_index<index, C>::container_type container_type;

  return static_cast<container_type&>(p_container).value;
}
```
### Implementation

Now that we have our `get` method, we can use it to extract the specific values. Also, remember that templates are a compile-time construct, so incorrectly assuming a type will yield a compile-time error (a helpful one at that!"). Here's the example use:
```cpp
#include <iostream>
#include <string>
#include "container.hpp"

// entry point
int main(int argc, const char* argv[]) {
  // create container<std::string, int, double, float>
  auto c = make_container(std::string("hello"), 32, 46.2, 0.6f);

  // explicit typed
  double d_value = get<2>(c);
  std::cout << "d_value: " << d_value << std::endl;

  // this generates a *compile-time* error
  // int i_value = get<0>(c); 
  // "No viable conversion from 'typename container_index<0, container<basic_string<char>, int, double, float> >::type' 
  // (aka 'std::__1::basic_string<char>') to 'int'"

  // print out results
  std::cout << "string: "   << get<0>(c)
            << ", int: "    << get<1>(c)
            << ", double: " << get<2>(c)
            << ", float: "  << get<3>(c) << std::endl;

  return 0;
}


Output:
d_value: 46.2
string: hello, int: 32, double: 46.2, float: 0.6
```
### _Final Thoughts_

Hopefully this walk-through has proven helpful if you are new to exploring the newer C++11 features. It's **_extremely_** important to note that I left out a lot of "conventional" C++ for the sake of clarity. I wouldn't suggest using the `container<T, Ts...>` for anything other than a guide. As mentioned previously, you should use [`std::tuple`](http://en.cppreference.com/w/cpp/utility/tuple) for serious use.

Admittedly, I still struggle a lot with C++ semantics, especially as it pertains to the newer features. This blog was a result of my struggle to find a clear set of examples for variadic templates, at least such that I felt comfortable using them. Currently, my plan was to use this feature to represent shader inputs, so if you needed to add a new input, the added `T` could propagate to any implementations, rather than be required to add support for the new type anywhere it was needed. Point being, don't limit the use of this amazing feature to an arbitrary type/value list!

All the code from the examples can be downloaded here: [variadic\_templates.tar.gz](https://drive.google.com/file/d/0BxxfE33wAHA7NXpnRHV5czZKTU0/edit?usp=sharing).

The specific version of the compiler I used for these examples:
```
Apple LLVM version 5.1 (clang-503.0.40) (based on LLVM 3.4svn)
Target: x86_64-apple-darwin13.1.0
Thread model: posix
```

I'm also nearly positive that Visual Studio 2013 will also compile without a problem.
