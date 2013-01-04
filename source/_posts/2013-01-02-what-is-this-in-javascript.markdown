---
layout: post
title: "What is 'this' in Javascript?"
date: 2013-01-02 13:44
comments: true
categories: [Fundamentals, Javascript]
---

Lately I've been writing a series of posts on the core differences and similarities between the Javascript and Ruby object models.
The idea is to explore the core fundamentals of the object models, since I'm a believer in really cementing the basics. It's tempting
to gloss over the basics, especially as developers since your world is absorbed in detail. But to use such awesome tools like Ruby or Javascript
effectively, you really need to understand some of the fundamental details of the language. One of the things that trumped me when first looking at
Javascript was some confusion surrounding the `this` keyword. So I thought I'd jot down a kind of cheat sheet on the meaning of `this`.

<!-- more -->

If we take a Javascript function like:

``` javascript
function simple(name) {
  this.name = name;
}
```

What is the value of `this`? And what actually does `this` mean in Javascript? 

### The current context
 
The value of `this` in a function call depends on the *execution context* of the function. If we take the above execution of `simple()`,
`this` would refer to the global object and the `name` property would be added to that object. Every Javascript runtime has a global object, in a browser the global object is `window`,
so in this example, name would be a property of the `window` object:
 
``` javascript
simple("Adam");
console.log(window.name);
=> "Adam"
```

However for method functions, that is, invoking functions that are referenced by object properties:
 
``` javascript
simple_obj = {
  simple: function(name) {
    this.name = name;
  };
}
 
simple_obj.simple("Adam");
console.log(simple_obj.name);
=> "Adam"
```
 
the value of `this` is bound to `simple_obj` at invocation. In other words, since `simple_obj` is the *receiver* of the `simple` function call, the *current context*
is now `simple_obj`, not the global object.
 
*The basic rule is that `this` binds to the global object, except when the function is invoked on an object, in which case `this` is bound to the object.*

However, there are some pretty unintuitive features of Javascript, and one of them is how `this` is bound in a nested function:
 
``` javascript
simple_obj = {
  simple: function(name) {
    this.name = name;
    log = function() {
      console.log("name was set to:" + this.name);
    }
    log();
  }
}
```
 
the value of `this` refers to the global object and is not bound to `simple_obj` like its parent. This is because `log` is not invoked on a *method reciever*.
This seems to be at odds with how scope works in Javascript, where you might think the value of `this` comes from the parent function through the
*closure*, but consider the rule is that a function invocation requires a *reciever* for `this` to be bound, no matter how the function is scoped. That is why you see code like:

``` javascript
simple_obj = {
  simple: function(name) {
    this.name = name
    var that = this;
    log = function() {
      console.log("name was set to: " + that.name)
    }
    log();
  }
}
```

The `that` local variable is accessed through the closure, and hence the inner function is able to access the context of its parent function. As a Javascript developer,
you would see this pattern used *a lot*.

Note that if you have an independent function and you add it as an object property later, any reference to `this` inside the function will still be bound to the object that it's called on:

``` javascript
var simple = function(name) {
  this.name = name;
}
obj = {
  f: simple
};

obj.f("Adam");
console.log(obj.name);
```

This emphasizes the *late binding* nature of Javascript.

### What about using the new operator?

The `new` operator *overrides* how `this` is usually bound. When you invoke a function with `new`, you're invoking it as a constructor function, creating a *new object*
and binding it to `this`:

``` javascript
function Simple(name) {
  this.name = name;
}

simple_obj = new Simple("Adam");
console.log(simple_obj.name);
=> "Adam"
```
So we can adjust our rule to be:
*`this` is bound to the global object, except when function is invoked on an object, in which case `this` is bound to the object. The `new` operator overrides this rule
by binding `this` to a new object.*

Forgetting to use the `new` operator, and `this` will be bound to the global object.

You can read more about how Javascript (and Ruby) objects are created [here](/blog/2012/12/20/a-tale-of-two-object-models-javascript-and-ruby-part-2/).

### Call and Apply

The `Function.prototype.call` and `Function.prototype.apply` functions allow you to pass as an argument the object you wish `this` to be bound to.

``` javascript
function simple(name) {
  this.name = name;
}

obj = {};

simple.call(obj);
console.log(obj.name);
```

This allows you to take a function and explicitly determine which object `this` is bound to. *Note the call and apply work the same way, except that
call accepts an array of function parameters as an argument, whereas with apply you pass those arguments individually.*
