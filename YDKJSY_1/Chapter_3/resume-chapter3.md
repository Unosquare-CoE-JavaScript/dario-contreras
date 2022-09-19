# Chapter 3: Digging to the Roots of JS 

## Iteration 

Since programs are essentially built to process data, the patterns used to step through the data have a big impact on the program's readability. 

The iterator pattern has been around for decades, and suggests a "standarized" approach to consuming data from a source, one chunk at a time. 

Imagine a data structure that represents a relational database `SELECT`query, which typically organizes the results as rows. If this query had only one or a couple of rows, you could handle the entire result set at once, and assign each row to a local variable, and perform whatever operations on that data that were appropriate. 

But if the query has 100 or 1,000 (or more!) rows, you'll need iterative processing to deal with this data (typically, a loop).

The iterator pattern defines a data structure called an "iterator" that has a reference to an underlying data source (like the query result rows), which exposes a method like `next()`. Calling `next()` returns the next piece of data (i.e., a "record" or "row" from a database query).

You don't always know how many pieces of data that you will need to iterate through, so the pattern typically indicates completion by some special value or exception once you iterate through the entire set and go past the end. 

The importance of the iterator pattern is in adhering to a standard way of processing data iteratively, which creates cleaner and easier to understand code, as opposed to having every data structure/source difine its own custom way of handling its data. 

After many years of various JS community efforts around mutually agreed-upon iteration techniques, ES6 standardized a specific protocol for the iterator pattern directly in the language. The protocol defines a `next()` method whose return is an object called an iterator result; the object has `value`and done properties, where `done`is a boolean that is `false`until the iteration over the underlying data source is complete. 

## Consuming Iterators 

With the ES6 iteration protocol in place, it's workable to consume a data source one value at a time, checking after `next()` call for `done` to be `true`to stop the iteration. 

Bare in mind that this approach is rather manual, so ES6 also included several mechanisms for standardized consumption of these iterators. 

One such mechanism is the `for...of` loop:

    //given an iterator of some data source: 
    var it = /* .. */;

    //loop over its results one at a time
    for (let val of it) {
        console.log(`Iterator value: ${ val }`);
    }

    //Iterator value: ..
    //Iterator value: ..
    //..

Another commonly used mechanism is the `...` operator. This operator has two symmtrical forms: spread and rest (or gather). The spread form is an iterator-consumer.  

To spread an iterator, you have to have something to spread it into. There are two possibilities in JS: an array or an argument list for a function call. 

An array spread: 

    // spread an iterator into an array, 
    // with each iterated value occupying 
    // an array element position. 

    var vals = [ ...it ];

A function call spread: 

    // spread an iterator into a function, 
    // call with each iterated value 
    // occupying an argument position. 
    doSomethingUseful( ...it );

## Iterables 

The iterator-consumption protocol is technically defined for consuming iterables; an iterable is a value that can be iterated over. 

The protocol automatically creates an iterator instance from an iterable, and consumes just that iterator instance to its completion. This means a single iterable could be consumed more than once; each time, a new iterator instance would be created and used. 

So where do we find iterables? 

ES6 defined the basic data structure/collection types in JS as iterables. This includes strings, arrays, maps, sets, and others. 

Consider: 

    // an array is an iterable 
    var arr = [ 10, 20, 30 ];

    for (let val of arr) {
        console.log(`Array value: ${ val }`);
    }

    // Array value: 10
    // Array value: 20
    // Array value: 30

We can also shallow-copy an array using iterator consumption via the `...` spread operator: 

    var arrCopy = [ ...arr ];

We can also iterate the characters in a string one at a time: 

    var greeting = "Hello world!";
    var chars = [ ...greeting ];
    
    chars;
    // [ "H", "e", "l", "l", "o", " ",
    //   "w", "o", "r", "l", "d", "!",]

A `Map` data structure uses objects as keys, associating a value with that object. `Maps` have a different default iteration than seen here, in that the iteration is not just over the map's values but instead its entries. An entry is a tuple (2-element array) including both a key and a value. 

Consider: 

    // given two DOM elements, `btn1` and `btn2`

    var buttonNames = new Map();
    buttonNames.set(btn1, "Button 1");
    buttonNames.set(btn2, "Button 2");

    for (let [btn,btnName] of buttonNames) {
        btn.addEventListenet("click", function onClick(){
            console.log(`Clicked ${ btnName }`);
        });
    }

In the `for..of` loop over the default map iteration, we use array destructuring syntax `[btn,btnName]` to break down each consumend tuple into the respective key/value pairs. 

If we want to consume only the values of the above `buttonNames` map, we can call `values()` to get a values-only iterator: 

    for (let btnName of buttonNames.values()) {
        console.log(btnName);
    }

    // Button 1
    // Button 2 

If we want the index and value in an array iteration, we can make an entries iterator with the `entries()` method: 

    var arr = [ 10, 20, 30 ];

    for (let [idx,val] of arr.entries()) {
        console.log(`[${ idx }]: ${ val }`);
    }

    // [0]: 10 
    // [1]: 20 
    // [2]: 30

Almost all iterables in JS have three iterator forms available: keys-only (`keys()`), values-only(`values()`), and entries(`entries()`).

## Closure 

Closure is when a function remembers and continues to access variables from outside its scope, even when the function is executed in a different scope. 

1. Closure is part of the nature of a function. Objects don't get closures, functions do. 
2. To observe a closure, you must execute a function in a different scope that where that function was originally defined. 

Consider: 

    function greeting(msg) { 
        return function who(name) {
            console.log(`${ msg }, ${ name }!`);
        }
    }

    var hello = greeting("Hello"); 
    var howdy = greeting("Howdy"); 

    hello("Kyle");
    // Hello, Kyle!

    hello("Sarah");
    // Hello, Sarah!

    howdy("Grant");
    // Howdy, Grant!

First, the `greeting(..)` outer function is executed, creating an instance of the inner function `who(..)`; that function closes over the variable `msg`, which is the parameter from the outer scope of `greeting(..)`. When that inner function is returned, its reference is assigned to the `hello` variable in the outer scope. Then we call `greeting(..)` a second time, creating a new inner function instance, with a new closure over a new `msg`, and return that reference to be assigned to howdy. 

When the `greeting(..)` function finishes running, normally we would expect all of its varibales to be garbage collected. We'd expect each `msg` to go away, but they don't. the reason is closure. Since the inner function instances are still alive (assigned to `hello` and `howdy`, respectively), their closures are still preserving the `msg` variables. 

These closures are not a snapshot of the `msg` variable's value; they are a direct link and preservation of the variable itself. The means closure can observe or make updates to these variables over time. 

    function counter(step = 1) {
        var count = 0;
        return function increaseCount() {
            count = count + step;
            return count; 
        }
    }

    var incBy1 = counter(1);
    var incBy3 = counter(3);

    incBy1();           // 1
    incBy1();           // 2

    incBy3();           // 3
    incBy3();           // 6
    incBy3();           // 9

Each instance of the inner `increaseCount()` function is closed over both the `count` and `step` variables from its outer `counter(..)` function's scope. `step` remains the same over time, but `count` is updated on each invocation of that inner function. 

Closure is most common when working with asynchronous code, such as with callbacks. Consider: 

    function getSomeDate(url) {
        ajax(url, function onResponse(resp){
            console.log(
                `Response (from ${ url }): ${ resp }`
            );
        });
    }

    getSomeData("https://some.url/wherever");
    // Response (from https://some.url/wherever): ... 

The innger function `onResponse(..)` is  closed over `url`, and thus preserves and remembers it until the Ajax call returns and executes `onResponse(..)`. Even though `getSomeData(..)` finishes right away, the `url` parameter variable is kept alive in the closure for as long as needed. 

It's not neccesary that the outer scope be a function, just that there be at least one variable in an outer scope accessed from an inner function: 

    for(let [idx, btn] of buttons.entries()) {
        btn.addEventListener("click", function onClick(){
            console.log(`Clicked on button (${ idx })!`);
        });
    }

## `this` Keyword 

As discussed previously, when a function is defined, it is attached to its enclosing scope via closure. Scope is the set of rules that controls how references to variables are resolved.  

Functions also have another characteristic besides their scope that influences what they can access. This characteristic is best described as an execution context, and it's exposed to the function via its `this` keyword. 

Scope is static and contains a fixed set of variables availavle at the moment and location you define a function, but a function's execution context is dynamic, entirely dependent on how it is called. 

`this` is not a fixed characteristic of a function based on the function's definition, but rather a dynamic characteristic that's determined each time the function is called. 

One way to think about the execution context is that it's a tangible object whose properties are made available to a function while it executes. 

Consider: 

    function classroom(teach) {
        return function assigment() {
            console.log(
                `${ teacher } says to study ${ this.topic }`
            );
        };
    }

The inner `study()` function references `this`, which makes the function a `this`-aware function. In other words, it's a function that is dependent on its execution context. 

Now consider: 

    var homework = {
        topic: "JS", 
        assignment: assignment
    };

    homework.assignment();
    //Kyle says to study JS 

A copy of the assignment function reference is set as a property on the `homework` object, and then it's called as `homework.assignment()`. That means the `this` for that function call will be the `homework` object. Hence, `this.topic` resolves to "JS".

Lastly: 

    var otherHomework = {
        topic: "Math"
    },

    assignment.call(otherHomework);
    //Kyle says to study Math

The `call(..)` method takes an object to use for setting the `this` reference for the function call. The property reference `this.topic` resolves to "Math".

The benefit of `this`-aware functions is the ability to more flexibly re-use a single function with data from different objects. A function that has dynamic `this` context awareness can be quite helpful for certain tasks. 

## Prototypes 

A prototype is a characterisitc of an object, and specifically resolution of a property access. 

A prototype is a linkage between two objets. This prototype linkage occurs when an object is created; it's linked to another object that already exists. 

A series of objects linked together via prototypes is called the "prototype chain".

The purpose of this prototype linkage is so that accesses against B for properties/methods that B does not have, are delegated to A to handle. Delegation of property/method access allows two (or more!) objects to cooperate with ech other to perform a task. 

Consider defining an object as a normal literal: 

    var homework = {
        topic: "JS"
    };

The default prototype linkage of `homework` connects to the `Object.prototype` object, which has common built-in methods on it like `toString()` and `valueOf()`, among others. 

    homework.toString();        // [object Object]

`homework.toString()` works even though `homework` doesn't have a `toString()` method defined; the delegation invokes `Object.prototype.toString()` instead. 

## Object Linkage 

To define an object prototype linkage, you can create object using the `Object.create(..)` utility: 

    var homework = {
        topic: "JS"
    };

    var otherHomework = Object.create(homework);

    otherHomework.topic; // "JS"

The first argument to `Object.create(..)` specifies an object to link the newly created object to, and then returns the newly created and linked object. 

Delegation through the prototype chain only applies for accesses to lookup the value in a property. If you assign to a property of an object, that will apply directly to the object regardless of where that object is prototype linked to.

Consider: 

    homework.topic; 
    // "JS" 

    otherHomework.topic;
    // "JS"

    otherHomework.topic = "Math";
    otherHomework.topic;
    // "Math"

    homework.topic; 
    // "JS" -- not "Math"

The assigment to topic creates a property of that name directly on `otherHomework`; there's no effect on the `topic` property on `homework`. The next statement then accesses `otherHomework.topic`, and we see the non-delegated answer from that new property: "Math".

## `this` Revisited 

One of the main reasons `this` supports dynamic context based on how the function is called is so that method calls on objects which delegate through the prototype chain still maintain the expected `this`.

Consider: 

    var homework = {
        study() {
            console.log(`Please study ${ this.topic }`);
        }
    };

    var jsHomework = Object.create(homework);
    jsHomework.topic = "JS"; 
    jsHomework.study();
    // Please study JS

    var mathHomework = Object.create(homework);
    mathHomework.topic = "Math";
    mathHomework.study();
    // Please study Math

Unlike many other languages, JS's `this`being dynamic is a critical component of allowing prototype delegation, and indeed `class`, to work as expected! 