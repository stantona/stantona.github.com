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

In Javascript, you can create objects very easily using the literal method:

``` javascript
simple_obj = {
  name: "Adam"
}
```

But you can also create new objects using functions and the `new` operator:

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
it would be pretty cumbersome to define them using the literal method over and over (which I've actually seen in a pretty horrendous codebase). 
Constructor functions provide a way to define behaviour which is shared
by all new objects that are created by that function. This shared behaviour is made possible by the `Function.prototype` property.
Javascript is a classless language, but the prototype property on a function the closest thing to any notion of classes. But the prototype *is not a class*
but an ordinary object, and so you can mold this object by adding properties:

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
Every new object created from `Simple` adopts the `sayHi` method function. This is the idea of *prototypal inheritance*, sharing behaviour and creating an inheritance
chain. The inheritance chain is a linked list of objects and method lookups follow this chain until the end of the line. You can get the prototype of an object by doing:

``` javascript
Object.getPrototypeOf(simple_obj);
=> { sayHi: [Function]}
```

We will talk about inheritance and method lookups in the next post since this is a pretty important part of both object models.

An interesting note to consider is that creating an object using constructing functions entails creating a number of other objects:

* The constructor function (since functions are objects)
* (At least) one prototype object (which could in turn be created by other constructor functions)
* The object itself (created from the constructor function), with its internal prototype referencing the constructor's prototype

A lot of objects are at play when creating an object, emphasizing that objects are ubiquitous, even during the creation of other objects.

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

Classes are a central concept in Ruby allowing you to create *classes* of objects. But a huge part in understanding Ruby's object model is to embrace the idea that
clasess are also objects. This way of thinking may not come naturally if you're from a Java or C++ background. 

Classes are objects and Ruby has special syntax for creating new class objects. But it's no wonder that you can
also do this:

``` ruby
Simple = Class.new do
  def initialize
    @name = "Adam"
  end
end
```

This also creates a new class object, referenced by `Simple` which is a constant, passing as a parameter to `new` a block that provides the method definitions.
This reinforces the idea that instances of Classes are really just objects, that can be created like any other object.
Furthermore, it's no surprising that when you define a class (or
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

This shows that even though on the surface Ruby classes are similar to classes in C++ or Java, they are really just objects that create other objects with
a *class* of behaviour. I don't think it's unreasonable to also think of them as *factory* objects, except that the factories define the behaviour
of the objects that are created (through method definitions).

### Summary

Javascript and Ruby provide a similar method for creating objects. They both require the interaction of other objects to create a new object, emphasizing
how ubiquitous objects are in both languages. The important difference
is that the type of objects and interactions involved is different in both langagues. Ruby uses
objects inherited from `Class` to create new objects, whereas JS can either define literal objects, or invoke constructor functions (which
are in turn objects themselves) to create new objects.

Javascript allows the creation of new objects with shared behaviour via `Function.prototype`, the idea that new objects reference the prototype object of the constructor function,
thus inheriting its behaviour. In Ruby, this is achieved through class objects and method definitions. In the next part we will take a look at method lookups and inheritance and
see how it works in both languages.



