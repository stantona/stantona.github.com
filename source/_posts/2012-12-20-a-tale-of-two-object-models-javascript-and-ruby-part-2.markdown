---
layout: post
title: "A tale of two object models: Javascript and Ruby (Part 2)"
date: 2012-12-20 14:25
comments: true
categories: [Ruby, Javascript, Fundamentals]
---

This is a continuation of a series of posts that explore the differences and similarities of the Javascript and Ruby object models.
My first post introduced the core of both object models as a repository of references to other objects.
In this post, I want to expand on some of the concepts introduced in my previous post by comparing how objects are created.

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

Since this is a function, you can do whatever you need to do to initialize the object. In the sample we're just assigning properties. This is loosely similar to constructors
of other languages, or the `initialize` method in Ruby (which we'll get to in a bit).

The following describes what the `new` operator does:

* creates a new object
* sets the `constructor` property of the object to the constructor function that created it (`Simple` in the above example)
* binds that object to `this`
* executes the constructor function.
* returns the new object

I think of the `new` operator as *overriding* how `this` is normally bound during a function invocation. When functions are invoked without `new, `this` is either bound to the global
object if not invoked on a method receiver, or to the *receiver* object for method functions. With `new`, `this` is bound to a brand new object.
If you forget to use `new` when invoking `Simple` above, `this` will be bound to the global object (most likely `window` in a browser) and the `name`
property will be added to that.

So why bother using constructor functions if the literal method will suffice? Constructor functions provide a way to define behaviour which is shared
by all new objects through the `Function.prototype` property.
Javascript is a classless language, but the prototype property on a function  the closest thing to any notion of classes. By default, `prototype` references an `Object`, so you can
mold this object as you see fit:

``` javascript
function Simple() {
  this.name = "Adam";
}

Simple.prototype.sayHi = function() {
  return this.name + " says hi!";
}
```
Every new object created from `Simple` adopts the sayHi method function. This is the idea that new objects extend a previously created object, sharing behaviour and creating an inheritance
chain. Method lookups follow this chain until the end of the line. We will talk about inheritance and method lookups in the next post.

Literal objects are always an `Object` and hence inherit behaviour from `Object.prototype`. You have *little* control over its prototype hence the disadvantage of using literal objects.

In the first part I mentioned that Javascript objects are a repository of properties that reference other objects. Well there are two properties that
are always set during the creation of an object, and that is `constructor` and `__proto__`. *Note that __proto__ is a non-standard property which references the internal
prototype object*. `__proto__` provides a way of peeking at the internal prototype object. It's not read only which makes its potential use interesting. `Constructor` is a reference
to the function that created it.

``` javascript
employee.constructor;
=> [Function: Employee]
employee.__proto__.constructor;
=> [Function: Person]
```

An interesting note to consider is that creating an object using constructing functions entails creating at least three objects:

* The constructor function
* (At least) one prototype object (which could in turn be created by other constructor functions)
* The object itself (created from the constructor function)

This demonstrates Javascript's object orientated nature where creating an object requires an interaction between other objects which also need to be created.

### How are Ruby objects created?

In part one, I mentioned that Ruby objects are created by calling the `new` method on a class object that defines the behaviour you wish to give your new object:

``` ruby
class Simple
  def initialize
    @name = "Adam"
  end
  attr_accessor :name
end

simple = Simple.new
```
The important distinction is that `new` is a method of `Class`, and not a keyword operator as it is in Javascript.

Like JS, `new` creates a brand new object and binds it to the current context, or `self`, Ruby's equivalent to Javascript's `this`. Ruby also creates a reference to the
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
instance variables, reminiscent of how you would use Javascript constructor functions.

### More on Ruby classes

As mentioned in part one, classes in ruby are also objects. It's no wonder that you can also create classes like this:

``` ruby
Simple = Class.new do
  def initialize
    @name = "Adam"
  end
end
```

This creates a new class object, passing as a parameter to `new` a block that provides the method definitions. A new class is created
and is referenced by `Simple`, which is a Ruby constant. There's also no reason you can't set a class to a standard variable. There is no difference beteween
this definition and the former definition,
it's just that ruby gives you a special syntax creating class objects that is reminiscent of Java or C++. Furthermore, it's no surprising that when you define a class (or
*open* a class in Ruby parlance):

``` ruby
class Simple
  puts self.class
end

=> Class
```

`self` is bound to the class object referenced by `Simple` when used directly inside the class definition. This shows Ruby's Object Orientated nature, the fact that classes
are created like any other object at runtime.

Classes and Modules (the parent class of `Class`) are what give other objects their behaviour by allowing you to define methods.



