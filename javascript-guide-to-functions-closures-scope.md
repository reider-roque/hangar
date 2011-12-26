<!--
~~~
title: "The Javascript Guide to Objects, Functions, Closures and Scope"
slug: /javascript-guide-to-objects-functions-prototpyes-closures-scope
date: 2011-12-06
publish: no
tags: [javascript]
~~~
-->

#The JavaScript guide to Objects, Functions, Prototypes, Closures and Scope

Throughout my time writing Javascript code, I've come to realize that while I love the language to bits, it is a little difficult to understand. A lot of people have attributed this to its  (admittedly not so great) design, or its obvious deviations from the paths well worn by other languages. Either way, understanding a few simple truths can go a long way with JS. 

Let's dive in with objects and variables. 

    var name = 'zaphod';
    var age = 42;

That was easy. The `var` keyword restricts the scope of the variables and prevents them from going global without us knowing, but we'll talk about that a little later. 

Now how about

	 var person = {name: 'zaphod', age: 42};

That's a little more interesting. What we're doing here is creating one variable, `person`. That variable contains the `name` and the `age` in a simple key value store… a map, if you will. Interestingly, it turns out that all Javascript objects can be thought of this way; they're all essentially key-value maps, or dictionaries. 

Accessing the data in these objects is easy:

	 alert(person.name);
	 alert(person.age);

What's more interesting is that this is equivalent to 

	 alert(person['name']);
	 alert(person['age']);

This is the first 'aha' moment for most people, because it starts to give you a hazy idea of power that this approach gives you. More on that soon, though. Let's move on. 

    var sayHello = function(){
        alert('Hello.');    
    };

There's a load of things to learn from the three lines above. First, this looks suspiciously like the way we normally define objects. *That's because it is.* Functions in JS are just glorified objects. Truth be told, they aren't even very glorified - they're just objects which can be 'called'.

    var sayHello = function(name, age) {
        return "Hello, I'm " + name + " and I'm " + age + "years old.";
    }

Simple enough. They take arguments and they return values. Better still, we could write: 

    var sayHello = function(person) {
        return "Hello, I'm " + person.name + " and I'm " + person.age + "years old.";
    }

So far so good. Call the function with your person, and you'd get what you expect. 

    sayHello(person);

So way to *call* a function *object* is to use `()`, possibly with arguments inside. 

But if functions are just objects themselves, then what's to stop us from doing this?

    person.doSomething = sayHello;

So `person.doSomething(person)` should give the same result. That's good. But the `person` seems written there too many times, though. If we're calling the function off a person, why pass the person in again? Let's try something else:

    var sayHello = function() {
        return "Hello, I'm " + this.name + " and I'm " + this.age + "years old.";
    }

So here's our first look at **scope**. Scope is the environment, or world, that the function executes in. In JS the default scope for executing functions is the `Window` object of that particular page, which is a browser level object that represents the window (or rather tab, these days) that the page is on. 

So that means running `sayHello()` (by itself) now will give you an error - `this.name` will refer to the current tab object's name, but I don't think your tab will have an `age` property. That's something to watch out for; if the function only referred to `this.name`, it would have always run, but probably not with the results you expect. 

This should give us what we want:
    
    person.doSomething = sayHello;
    person.doSomething();

Now when `doSomething` runs, it executes in the scope of `person`. So `this.name` and `this.age` will use that's person object's data. 

We still haven't modified `sayHello` itself, though. So calling `sayHello()` will still give the error. `doSomething` only has a *reference* to the same function that `sayHello` has. You can confirm this by subsequently setting `sayHello` to something else. `person.doSomething` will continue to work as expected. 

Here's another interesting piece of code:

    var zaphod = {name: 'Zaphod', age: 42};
    var marvin = {name: 'Marvin', age: 420000000000;}

    zaphod.sayHello = function(){
        return "Hi, I'm " + this.name;
    }
    marvin.sayHello = zaphod.sayHello;

So `zaphod.sayHello()` is going to give you "Hi, I'm Zaphod". Trick question: *What is `marvin.sayHello()` going to return?* 

That's the next thing to remember about scope. It's always *runtime*. This means that the scope of a function is always the context in which it *executes*, not the context in which it was *defined*. So `marvin.sayHello()` is going to return "Hi, I'm Marvin."

We already saw this in the last example: the function was actually defined in the context of `Window` when there was no obvious or explicit scope. This one makes it a lot more clear. 

Let's use what we've looked as so far to build a simple calculator. 

    var plus = function(x,y){ return x + y };
    var minus = function(x,y){ return x - y };

    var operations = {
        '+': plus,
        '-': minus
    };

The more functional way to use this would probalby be someting like this:

    var calculate = function(x, y, operation){
        return operations[operation](x, y);
    }
    
    calculate(38, 4, '+');
    calculate(47, 3, '-');
    
This example also shows how absurdly easy it is to implement some patterns like the [strategy pattern](http://en.wikipedia.org/wiki/Strategy_pattern) in JavaScript. While I've never liked implementing patterns for their own sake, this is still something to remember. 

On the other hand, let's try something a little more object-oriented. While most 'object oriented' langauges today use class based inheritance, JavaScript uses prototypes. There is no class construct here, but it's still a very powerful langauage. Let's see how they work.

    var Problem = function(x, y){
        this.x = x;
        this.y = y;        
    }
    
So here we have a function that takes two parameters, `x` and `y` and stores them. Now keep in mind that since functions are simply objects, you can easily do this:

    var problem1 = new Problem(4, 5);
    alert(problem1.x);
    alert(problem1.y);

The `new` keyword is essential here because it tells the interpreter that you're using `Problem` not as a function, but as a constructor instead. Leaving it out will simply execute the `Problem` function, which returns nothing, so `problem1` will have no value. The `new` keyword gives us a copy of the function object after running it, while preserving the original intact (so we can use it to make more `problem` objects).
    
Now we'd like to have our `Problem` capable of solving itself, so let's do this:

    Problem.prototype.operations = {
        '+': function(x,y){ return x + y },
        '-': function(x,y){ return x - y }
    };
    
    Problem.prototype.calculate = function(operation){
        return this.operations[operation](this.x, this.y);
    };
    
    problem1.calculate('+');
    problem1.calculate('-');
    
So here's our first look at the JavaScript `prototype`. Although it's one of the cornerstones of the language, it's possible to write JS for years and not come across any reason to use it. Like most things, though, understanding it is essential to mastery. 

In JS, each and every object has one built-in property called the `__proto__`. This property refers to another object which is considered the prototype for this object, a sort of *parent* or *mould* or *original* from which this object was created. This is a very useful link because it means that all objects have the properties and methods of their prototype. 

Conversely, modifying a prototype to add new features to it will result in that new feature being available to all the objects which have that prototype. 

The prototype model in JS works similar to the class model in other languages, in the sense that when a method is not found in the class definition of an object, it's superclass is then checked, and so on. The same process takes place here - first the object is checked. If the method or property is not found, it's `__proto__` is checked, then it's prototype's `__proto__` and so on, until the interpreter hits the last object on the chain (`__proto__` is `null`).

You'll notice though, that we haven't touched `__proto__` anywhere in our code. That's because it doesn't make sense to and is a *very bad idea* to do so. Instead JS lets you use the `prototype` object on the function that you're using as a constructor (`Problem`, in this case) to specify the behaviour of the prototypes of the constructed objects.
    
 































