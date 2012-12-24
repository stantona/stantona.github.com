---
layout: post
title: "A tale of two object models: Javascript and Ruby (Part 2)"
date: 2012-12-20 14:25
comments: true
categories: [Ruby, Javascript, Fundamentals]
---

This is a continuation of a series of posts that explore the differences and similarities of the Javascript and Ruby object models.
My first post introduced the core of both object models as essentially a repository of references to other objects.
In this post, I want to expand on some of the concepts introduced in my previous post by comparing how objects are created.

## How are Javascript objects created?

In my previous post I showed how Javascript objects can be created using the literal method but
you can also create new objects by using functions, prepending the call using the `new` keyword:

``` javascript
function Simple() {
  this.name = "Adam"
}

simple_obj = new Simple();
```

The function above is a *constructor* function (since it is called with the new operator) because its purpose is to create new objects.

Notice you can assign properties inside that function, performing any required initialization work. This is loosely similar to constructors
of other languages, or the `initialize` method in Ruby (which we'll get to in a bit).

I think of the `new` operator as `overriding` how `this` is usually bound during a function invocation. Normally, `this` is either bound to the global
object, or to the *receiver* object for method functions. With `new`, `this` is bound to a brand new shiny object. But it does a little bit more than that:

* creates new object
* sets the constructor property of the object to the constructor function
* binds that object to `this`
* executes the constructor function.
* returns the new object

So why bother using constructor functions if the literal method will suffice? Constructor functions provide you a way to configure the prototype object
from which new objects are created. This is the idea of prototypal inheritance a central feature to the language. Each object has a prototype which references
another object, forming a finite prototype chain. Property lookups fall back to the prototype if the property can
not be found. This is the fundamental way of sharing behaviour for multiple instances of objects.

``` javascript
function Person(name) {
  this.name = name;
}
function Employee() {
  this.position = "Developer";
}
Employee.prototype = new Person("Adam");
employee = new Employee();
employee.name;
=> "Adam";
employee.position;
=> "Developer";
```

Literal objects
are always an `Object` and hence inherit from `Object.prototype` and this is no good if you plan on creating many instances of objects that have
the same behaviour. With constructor functions, you can specify as a property the prototype object by which all created objects adopt:


In the first part I mentioned that javascript objects are a repository of properties that reference other objects. Well there are two properties that
are always set during the creation of an object, and that is `constructor` and `__proto__`. *Note that __proto__ is a non-standard property which references the internal
prototype object*.

``` javascript
employee.constructor;
=> [Function: Employee]
employee.__proto__.constructor;
=> [Function: Person]
```

So when you create a new employee, it creates a *brand new object* and sets the `__proto__` property to the value of the prototype property on the constructor function.

The prototype chain will be discussed in the next part when we talk about inheritance and method lookups.

### How are Ruby objects created?

As pointed out in part 1, Ruby objects are created by calling the `new` method of a class.

``` ruby
class Simple
  def initialize
    @name = "Adam"
  end
  attr_accessor :name
end

simple = Simple.new
```

This is the only way you can create user defined classes. Obviously Ruby gives you simple ways of creating strings and numbers.
Even ruby hashes, which look a lot like JS literal objects, are still Hash objects:

``` ruby
hash = { name: "Adam" }
```


