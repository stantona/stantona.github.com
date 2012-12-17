---
layout: post
title: "A tale of two object models: Javascript and Ruby (Part 1)"
date: 2012-12-12 23:22
comments: true
categories: [Javascript, Ruby, Fundamentals]
---

This past year I've been developing a web application with Ruby on Rails and javascript, and although javascript
is not a new language for me, I've had an amazing time learning Ruby and its powerful metaprogramming capabilities.
As I regularly switch between the two languages (as you do buidling web applications) I've recently found myself comparing the two
object models. I thought I would give my own personal take on the object models of
both languages, cementing the fundamental differences while demonstrating the similarities 

This is a loaded topic, so I thought I would start with the fundamentals and talk about what objects are exactly in these two languages.

## What are javascript objects?
Javascript objects are composite data structures comprised of properties.
You can create simple objects and write to properties like so:
``` javascript
var obj = {};
obj.name = "Adam"
```

is the same as:
``` javascript
var obj = {
  name: "Adam"
};
```
which is the same as:
``` javascript
var obj = {};
obj["name"] = "Adam";
```

The above examples demonstrate how objects are created using the literal technique (you can also create javascript objects using constructor functions, but we'll get to that in part two).
Literal objects are simple objects that you mold to your liking by setting properties. Since functions are also objects, you can define a property as a function
and execute it:
``` javascript
var obj = {};
obj.sayHi = function() {
  return "hi";
}
obj.sayHi(); // returns hi
```

Note the familiar `()` syntax after the function call: this means execute me like I'm a function. If you leave that out, you return the function
as if you're calling the name property shown above.

It's reasonable to think that javascript objects look a lot like Hashes, and if you were to compare JS objects to Ruby hash objects, they look syntactically similar.

``` ruby
hash = { name: "Adam" } #ruby hash
puts hash[:name] #prints "Adam"
```
``` javascript
var obj = { name: "Adam" } // js object
console.log(obj["name"]) //prints "Adam"
```

As you can see, JS objects appear like simple dictionaries, a repository of properties that reference other objects and you can access these properties directly on the object.
As a side note, various JS engines do not necessarily implement objects as hash tables under the hood. Check out the way [V8](https://developers.google.com/v8/design?hl=sv#prop_access)
implements property access.

## What are Ruby objects?
Where a JS object *is a collection* of properties that reference other objects, a ruby object *contains a hash* of instance variables that reference other objects. You can not access these
instance variables in the same way that you can access (read or write to) JS properties. Ruby enforces a level of data encapsulation which protects these variables.
Only when you are within the context of an object, which occurs during a method call, you can create or alter instance variables.

``` ruby
class Simple
  def var
    @var
  end
  def var=(val)
    @var = val
  end
simple_obj = Simple.new
```

Before we talk about the instance variable `@var`, it's a good time to discuss the `Simple` class above, because Ruby objects can not exist without classes.

### A (very) brief intro to Ruby classes
Ruby is a class based language, whereas JS is not (although you can achieve level of class like behaviour). In Ruby, every object has an associated class:

```
[1] pry(main)> 2.class
=> Fixnum
[2] pry(main)> simple_obj.class
=> Simple
```

You could say that the JS equivalent to this method is the `__proto__` property, even though this represents the prototype object from which the JS object was created.
This will be discussed in the length in part two.

Ruby classes are a place to define methods, which give objects their behaviour:

``` ruby Passing false excludes instance methods from ancestor classes
[1] pry(main)> Simple.instance_methods(false)
=> [:var, :var=]
```

Notice that the methods returned above are defined in our Simple class  These are the proverbial getters and setters that are familiar if you have used Java or C#. They provide
access to the `@var` instance variable that we can not access otherwise. In Ruby, we can simplify the Simple class code by calling the class method `attr_accessor` to define these 
getters and setters for us:

``` ruby
class Simple
  attr_accessor :var
end
```

If you repeat the `instance_methods` call on our new implementation:

```
[1] pry(main)> Simple.instance_methods(false)
=> [:var, :var=]
```
the results are the same as the original implementation. `attr_accessor` wrote these methods for us. This is a little bit of *metaprogramming* magic (which we will talk about in a future part).

In JS, there would be no need to create functions to do this, since properties on JS objects are publicly accessible.

In Ruby, classes (like almost everything) are objects and so we can call methods directly on them:

``` ruby
[1] pry(main)> Simple.is_a? Object
=> true
[2] pry(main)> Simple.class
=> Class
```

Notice that the class of Simple is Class. If you peek at the instance methods of Class:

``` ruby
[1] pry(main)> Class.instance_methods(false)
=> [:allocate, :new, :duplicate]
```
you will find the `new` method. This method is the mechanism to create a new object (covered in part two).

Classes are responsible for a lot of magic in Ruby, and we will cover Ruby classes extensively in future parts.

### Instance variables
So getting back to the `@var` instance variable. In the sample above, `@var` is accessible through the getters and setters:

``` ruby
[4] pry(main)> simple_obj.var
=> nil
[5] pry(main)> simple_obj.var = 4
[6] pry(main)> simple_obj.var
=> 4
```

Note that this appears similar to JS access to properties, but in fact, `var` is a method call on `simple_obj`, which is the method *receiver*. In the JS equivalent, var would be a property, not a method (function) call.

The @ notation basically means: *access this instance variable in the scope of the current context.*
The current context is the object referenced by `simple_obj`. The current context is an important concept in both JS and Ruby (which is where the `this` and `self` keywords come into play),
and we will cover this in part two when we look at object creation.

Remember when I said that instance variables are private to the object? Well this is not quite true. Ruby gives you a lot of power and has some interesting methods that allow you to peek inside the object:

``` ruby
[1] pry(main)> simple_obj = Simple.new
[2] pry(main)> simple_obj.instance_variables
=> []
```

At this point, no instance variables have been assigned to the object.

``` ruby
[3] pry(main)> simple_obj.instance_variable_set[:@var] = "hi!"
[4] pry(main)> simple_obj.instance_variables
=> [:@var]
```
Now that '@var' is assigned, see it in the second call.

``` ruby
[5] pry(main)> simple_obj.instance_variable_get[:@var]
=> "hi!"
```

## Summary

At their very core objects in JS and Ruby are simply repositories of references to other objects.

The essential difference is that JS does not restrict access to properties. The language itself does not support any data encapsulation on objects directly. That does not mean you can
not achieve this in other ways however through function scopes acting as modules.

Ruby on the other hand enforces a certain level of data encapsulation by not providing a simple way to access its internal instance variables. You have to enter the object's context through a method call
to operate on instance variables. However, we just demonstrated above that there are methods that allow you to peek into, change or get individual instance variables which break
this data encapsulation.

Javascript objects are quite simple especially when created using the literal method.
It's not until we look at object
creation in the next part that we see that JS objects can also be created via constructor functions. This opens up the concept of *prototypal inheritance*, which is javascript's inheritance mechanism. This is where the JS
object model starts to get a little more involved.

Ruby objects are  more involved since objects contain a reference to the class that created it. The object relies heavily on this relationship throughout its lifetime, since whenever a method is called, the object refers to its class (and potentially ancestor classes) to find and execute the method. 
Classes (and it's parent class, Module) are what make metaprogramming possible.

There is obviously *a lot* more to the object models of both languages, so I hope you will tune in to part two when I look at how objects are created in both languages.

### Further Resources

* [Javascript: The Good Parts (Douglas Crockford)](http://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742/ref=sr_1_1?ie=UTF8&qid=1355669318&sr=8-1&keywords=javascript+the+good+parts)
* Chapter 1 of [Metaprogramming Ruby (Paulo Perotta)](http://pragprog.com/book/ppmetr/metaprogramming-ruby)
* Episode 1 of [The Ruby Object Model and Metaprogramming (Dave Thomas)](http://pragprog.com/screencasts/v-dtrubyom/the-ruby-object-model-and-metaprogramming)


