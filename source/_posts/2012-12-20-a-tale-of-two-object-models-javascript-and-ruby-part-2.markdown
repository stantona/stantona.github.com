---
layout: post
title: "A tale of two object models: Javascript and Ruby (Part 2)"
date: 2012-12-20 14:25
comments: true
categories: [Ruby, Javascript, Fundamentals]
---

This is a continuation of posts that explore the differences and similarities of the Javascript and Ruby object models.
My first post introduced the core of both object models as a repository of references to other objects.
In this post, I want to expand on some of the concepts introduced in my previous post by comparing how objects are created.

<!-- more -->

## How are Javascript objects created?

In my previous post I showed how Javascript objects can be created using the literal method.
You can also create new objects using functions and the `new` operator:

``` javascript
function Simple() {
  this.name = "Adam"
}

simple_obj = new Simple();
```

The function above is a *constructor* function (when used with the new operator) because its purpose is to create new objects.

Since this is a function, you can do whatever you need to do to initialize the object. In the sample we're just assigning properties. In Ruby
you can do the same thing with the `initialize` instance method (which we'll get to in a bit).

The following describes what the `new` operator does:

* creates a new object
* sets the `constructor` property of the object to the constructor function that is passed to it (`Simple` in the above example)
* binds the new object to `this`
* executes the constructor function.
* returns the new object

I think of the `new` operator as *overriding* how `this` is normally bound during a function invocation. When functions are invoked without `new`, `this` is either bound to the global
object if not invoked on a method receiver, or to the *receiver* object for method functions. With `new`, `this` is bound to a brand new object.
If you forget to use `new` when invoking `Simple` above, `this` will be bound to the global object (most likely `window` in a browser) and the `name`
property will be added to that.

So why bother using constructor functions if literal definitions are straight forward and concise? If you needed to create multiple objects,
it would be pretty cumbersome to define them using the literal method over and over. Constructor functions provide a way to define behaviour which is shared
by all new objects that are created by that function. This shared behaviour is made possible by the `Function.prototype` property.
Javascript is a classless language, but the prototype property on a function the closest thing to any notion of classes. `Function.prototype` is just an object, so you can
mold this object by adding properties:

``` javascript
function Simple() {
  this.name = "Adam";
}

Simple.prototype.sayHi = function() {
  return this.name + " says hi!";
}

var simple = new Simple();
simple.sayHi();
=> "Adam says Hi!"
```
Every new object created from `Simple` adopts the sayHi method function. This is the idea of *prototypal inheritance*, sharing behaviour and creating an inheritance
chain when you consider that a prototype object can also have its own prototype object. Method lookups follow this chain of prototypes until the end of the line. 
We will talk about inheritance and method lookups in the next post.

Literal objects are always an `Object` and hence inherit behaviour from `Object.prototype`. You have *little* control over its prototype hence the disadvantage of using literal objects.

An interesting note to consider is that creating an object using constructing functions entails creating a number of other objects:

* The constructor function (since functions are objects)
* (At least) one prototype object (which could in turn be created by other constructor functions)
* The object itself (created from the constructor function), with its internal prototype referencing the constructor's prototype

A lot of objects are at play when creating an object, emphasizing the object orientation of Javascript.

### How are Ruby objects created?

In part one, I mentioned that Ruby objects are created by calling the `new` method on a class object:

``` ruby
class Simple
  def initialize
    @name = "Adam"
  end
  attr_accessor :name
end

simple = Simple.new
```
Classes provide a way of creating objects with defined behaviour (methods) via the `new` method.
The important distinction is that `new` is a method of `Class`, and not a keyword operator as it is in Javascript.

Like javascript, `new` creates a brand new object and binds it to the current context, or `self`, Ruby's equivalent to Javascript's `this`. Ruby also creates a reference to the
class that created it:

``` ruby
class Simple
  def initialize
    puts self.class
    @name = "Adam"
  end
end

simple = Simple.new
=> Simple
```

`new` then passes control to the `initialize` method defined on the class if it exists. This is where you can do any extra initialization work on the object, like setting
instance variables, similar to how you would use Javascript constructor functions.

### More on Ruby classes

As we just saw, classes provide the mechanism to create objects. Since classes in ruby are also objects, it's no wonder that you can 
also create classes like this:

``` ruby
Simple = Class.new do
  def initialize
    @name = "Adam"
  end
end
```

This creates a new class object, referenced by `Simple` which is a constant, passing as a parameter to `new` a block that provides the method definitions.
This reinforces the idea that instances of Classes are really just objects, that can be created like any other object.
It's just that ruby provides a special syntax for creating class objects which is familiar to users of Java or C#. Furthermore, it's no surprising that when you define a class (or
*open* a class in Ruby parlance):

``` ruby
class Simple
  puts self.class
end

=> Class
```

you enter the context of `Simple`, with `self` bound to that object (remembering that classes are objects) 
Once you have *opened* a class, you can write expressions and statements as you would anywhere else:

``` ruby
class Simple
  if rand(1..2) % 2 > 0
    attr_accessor :name
  end
end
```

With the above code, `Simple` may or may not get the `name` accessor.

This shows Ruby's Object Orientated nature, the fact that classes are created like any other object at runtime.

### Summary



