# Chapter 2

## Each File is a Program 

Almost every website (web application) you use is comprised of many different JS files (tyically with th e.js file extension). 

In JS, each standalone file is its own separte program. The reason this matters is primarily around error handling. Since JS treats files as programs, one file may fail and that will not necessarily prevent the next file from being processed. It's important to ensure that each file works properly, and that to whatever extent possible, they handle failure in other files as gracefully as possible. 

The only way multiple standalone .js files act as a single program is by sharing their state (and access to their public functionality) via the "global scope". 

Since ES6, JS has also supported a module format in addition to the typical standalone JS program format. Modules are also file-based. If a file is loaded via module-loading mechanism such as an import statement or a `<script type=module>` all its code si treated as a single module.

Though you wouldn't typically think about a module as a standalone program, JS does in fact still treat each module separately. Importing a module into another allows runtime interoperation between them. 

Regardless of which code organization pattern is used for a file, you should still think of each file as its own program, which may then cooperate with other programs to perform the functions of your overall application. 

## Values

The most fundamental unit of information in a program is a value. Values are data and they come in two forms in JS: **primitive** and **object**.

Values are embedded in programs using literals:

    greeting("My name is Kyle.");

In this program, the value "My name is Kyle." is a primitive string literal; strings are ordered collections of characters, usually used to represent words and sentences. 

Another option to delimit a string literal is to use the backtick character, but this has a behavioral difference. Consider:  

    var firstName = "Kyle"
    console.log(`My name is ${ firstName }.`)
    //My name is Kyle

The string delimited by backticks resolves the variable explresion (${}) to its current value. This is called interpolation. The backtick delimited string can be used without including interpolated expressions, but that defeats the whole purpose of that alternate string literal syntax. 

Other than strings, JS programs often contain other primitive literal values such as booleans and numbers: 

    while (false) {
        console.log(3.141592)
        }

`while` represents a loop type, a way to repeat operations while its condition is true.  

In this case, the loop will not run because we used the `false` boolean value as the loop conditional.

In addition to strings, numbers, and booleans, two other primitive values in JS are `null` and `undefined`. While there are differences between them, for the most part both values serve the purpose of indicating emptiness of a value. However, it's safest and best to use only `undefined` as the single empty value, even though `null` seems attractive in that it's shorter to type. 

    while (value != undefined) {
        console.log("Still got something!");
        }

The final primitive value is symbol, which is a special-purpose value that behaves as a hidden unguessable value. Symbols are almost exclusively used as special keys on objects: 

    hitchhikersGuide[ Symbol("meaning of life")];
    //42

Keep in mind that you won't encounter direct usage of symbols very often in typical JS programs. They're mostly used in low-level code such as in libraries and frameworks. 

## Arrays And Objects 

Besides primitives, the other value type in JS is an object value. 

Arrays are a special type of object that's comprised of an ordered and numerically indexed list of data: 

    names = ["Frank", "Kyle", "Peter", "Susan"];

    names.length;
    //4

    names[0];
    //Frank

    names[1];
    //Kyle

JS arrays can hold any value type, either primitive or object, even functions are values that can be held in arrays or objects.      

Objects are more general: an unordered, keyed collection of any various values. For example:    

    name = {
        first: "Kyle", 
        last: "Simpson", 
        age: 39, 
        specialties: ["JS", "Table Tennis"] 
    }

    console.log(`My name is ${ name.first}.`);


Here, `name` represents an object, and first represents the `name` of a location of information in that object (value colection). Another syntax option that accesses information in an object by its property/key uses the square-brackets `[]`, such as `name["first"]`.

## Value Type Determination 

For distinguishing values, the typeof operator tells you its built-in type, if primitive, or "object" otherwise:

    typeof 42;                               // "Number" 
    typeof "abc";                            // "String" 
    typeof true;                             // "boolean" 
    typeof undefined;                        // "undefined" 
    typeof null;                             // "object" -- oops, bug! 
    typeof { "a": 1 };                       // "object" 
    typeof [1,2,3];                          // "object" 
    typeof function hello(){};               // "function" 

Converting from one value type to another, such as from string to number, is called "coercion".

## Declaring and Using Variables 

In JS programs, values can either appear as literal values, or they can be held in variables; think of variables as just containers for values. 

Variables have to be declared to be used. There are various sintax forms that declare variables, and each form has different implied behaviors.  

Consider the statement:

    var name = "Kyle"; 
    var age";

The `var` keyword declares a variable to be used in that part of the program, and optionally allows initial value assignment. 

Another similar keywoed is `let`:

    let name = "Kyle" 
    let age;

The `let` keyword has some differences to `var`, with the most obvious being that `let` allows a more limited access to the variable than `var`. This is called "block scoping" as opposed to regular or function scoping.

Block-scoping is very useful for limiting how widespread variable declarations are in our programs, which helps prevent accidental overlap of their names. 

`var` is still useful in that it communicates "this variable will be seen by a wider scope". Both `var` and `let` can be appropriate in any given part of a program, depending on the circumstances. 

A third declaration form is `const`. It's like `let` but has an additional limitation that it must be given a value at the moment it's declared, and cannot be re-assigned a different value later. 

Consider: 

    const myBirthday = true 
    let age = 39; 

    if (myBirhday) { 
        age = age + 1;  // OK! 
        myBirthday = false; // Error! 
    }

The `myBirthday` constant is not allowed to be re-assigned. 

`const` declared variables are not "unchangeable", they just cannot be re-assigned. The best semantic use of a `const` is when you have a simple primitive value that you want to give a useful name to, such as using `myBirthday` instead of `true`.

## Functions

The word `function` has a veriety of meanings in programming. In JS,  we should consider `function` as a `procedure`. A procedure is a collection of statements that can be invoked one or more times, may be provided some inputs, and may give back one or more outputs. 

A classic function definition looks like:

    function awesomeFunction(coolThings){ 
        //.. 
        return amazingStuff; 
    }

This is called a function declaration because it appears as a statement by itself, not as an expression in another statement. 

The association between the identifier `awesomeFunction` and the function value happens during the compile phase of the code, before that code is executed. 

A function expression can be defined and assigned like this: 

    // let awesomeFunction = .. 
    // const awesomeFunction = .. 
    var awesomeFunction = function(coolThings) { 
        // .. 
        return amazingStuff;
    };

This function is an expression that is assigned to the variable `awesomeFunction`. Different from the function declaration form, a function expression is not associated with its identifier until that statement during runtime.

It's important to note that in JS, functions are values that can be assigned and passed around. In fact, JS functions are a special type of the object value type. Not all languages treat functios as values, but it's essential for a language to support the functional programming pattern, as JS does. 

JS function can receive parameter input, and also return values using the `return` keyword: 

    function greeting(myName){ 
        return `Hello, ${ myName }!`; 
    }

    var msg = greeting("Kyle");

    console.log(msg); // Hello, Kyle!

You can only `return` a single value, but if you have more values to `return`, you can wrap them up into a single object/array. 

Since functions are values, they can be assigned as properties on objects: 

    var whatToSay = { 
        greeting() {  
            console.log("Hello!"); 
        }, 
        question() {  
            console.log("What's your name?"); 
        }, 
        answer() {  
            console.log("My name is Kyle."); 
        } 
    };

    whatToSay.greeting();
    //Hello! 

In this snippet, each function can be called by accessing the property to retrieve the function reference value. 

## Comparisons 

Making decisions in programs requires comparing values to determine their identity and relationship to each other. JS has several mechanisms to enable value comparison, so let's take a closer look at them. 

## Equal...ish

In JS, an equility comparison sometimes intends exact matching, but other times the desired comparison is a bit broader, allowing closely similar or interchangeable matching, so we must be aware of the nuanced differences between an equality comparison and an equivalence comparison. 

The triple-equals `===` in JS are often described as a "strict equality" operator, but that's not exactly the case. 

While most values participating in an "===" equlity comparison will fit with that exact same intuition. Consider the following examples: 

    3 === 3.0;              // true 
    "yes" === "yes";        // true 
    null === null;          // true 
    false===false;          // true 

    42 === "42";            // false 
    "hello" === "Hello";    // false 
    true === 1;             // false 
    0 === null;             // false 
    "" === null;            // false 
    null===undefined;       // false 

    NaN === NaN;            // false 
    0=== -0;                // true  

In the case of `NaN`, the `===` operator lies and says that an occurrence of `NaN` is not equal to another `NaN`. Same lie for the comparison of 0 and -0. 

Since the lying avout such comparisons can be bothersome, it's best to avoid using "===" for them. For `NaN` comparisons, use the `Number.isNaN`(..) utility, which does not lie. For -0 comparison, use the `Object.is(..)` utility, which also does not lie, this could also be used for non-lying `NaN` checks, if you prefer. You could think of `Object.is(..)` as the "quadruple-equals", the really-really-strict comparison.  

Things get more complicated when we consider comparisons of object values. Consider: 

    [ 1, 2, 3 ] ==== [ 1, 2, 3 ];           // false 
    { a: 42 } === { a: 42 }                 // false 
    (x => x * 2) === (x => x * 2)           // false 

What's going on here? JS does not define `===` as structural equality for object values. Instead, `===` uses identity equality for object values. 

In JS, all object values are held by reference, and they are assigned and passed by reference-copy, and to our current discussion, are compared by reference equality. 

Consider: 

    var x = [ 1, 2, 3 ];

    // assignment is by reference-copy, so 
    // y references the *same* array as x, 
    // not another copy of it. 
    var y = x;
    
    y === x;             // true 
    y === [ 1, 2, 3 ];   // false 
    x === [ 1, 2, 3,];   // false 

In this snippet, `y === x` is true because both variables hold a reference to the same initial array. But the `=== [1, 2, 3]` comparisons both fail because y and x, respectively, are being compared to new different arrays `[1,2,3]`. The only thing that matters on object comparisons is reference identity. 

If you'd like to do structural equality comparisons in JS, you'll need to implement the checks yourself. 

## Coercive Comparisons 

Coercion means a value of one type being converted to its respective representation in another type. Coercion is core pillar of JS. 

The  `==` operator performs an equility comparison similarly to how the  `===`> performs it. In fact, both operators consider the type of the values being compared. And if the comparison is between the same value type, both  `==` and  `===`> do exactly the same thing. 

The difference is that, if the value types being compared are different,  `==` allows coercion before the comparison. In other words,  `==` allows type conversions first, and once the types have been converted to be the same on both sides, then  `==` does the same thing as  `===`>. The  `==` operator should be described as "coercive equality".

Consider: 

    42 == "42";      // true 
     1 == true;      // true

In both comparisons, the value types are different, so the  `==`  causes the  non-number values ( `42`  and  `true` ) to be converted to numbers ( `42`  and  `1` , respectively) before the comparisons are made. 

In the case where a numerical comparison is being made with two string values, an alphabetical comparison will be used: 

    var x = "10"; 
    var y = "9";

    x < y;          // true, watch out!

There's no way to get there relational operators to avoid coercion, other than to just never use mismatched types in the comparisons. Though the wiser approach is not to avoid coercive comparisons, but to embrace and learn their ins and outs. 

## How We Organize in JS 

Two major patterns for organizing code are used broadly across the JS ecosystem: classes and modules. These patterns are not mutually exclusive; many programs can and do use both. 

Being proficient in JS requires understanding both patterns and where they are appropriate (and not!).

## Classes

A class in a program is a definition of a "type" of custom data structure that includes both data and behaviors that operate on that data. Classes define how such a data structure works, but classes are not themselves concrete values. To get a concrete value that you can use in the program, a class must be instantiated one or more times. 

Consider: 

    class Page { 
        constructor(text){ 
            this.text = text; 
        } 

    print() { 
        console.log(this.text); 
        } 
    }

    Class Notebook {  
        constructor() {  
            this.pages = []; 
        }

        addPage(text) { 
            var page = new Page(text); 
            this.pages.push(page); 
        }

        print() { 
            for (let page of this.pages) { 
                    page.print(); 
            } 
        } 
    }

    var mathNotes = new Notebook(); 
    mathNotes.addPage("Arithmetic: + - * / ..."); 
    mathNotes.addPage("Trigonometry: sin cos tan ..."); 

    mathNotes.print(); 
    //..

In the `Page` class, the data is a string of text stored in a ``this.text`` member property. The behavior is `print()`, a method that dumps the text to the console. 

For the `Notebook` class, the data is an array of `Page` instances. The behavior is `addPage(..)`, a method that instantiates `new Page` pages and adds them to the list, as well as `print()`.

`mathNotes = new Notebook()` creates an instance of the `Notebook class`, and `page = new Page(text)` is where instances of the `Page class` are created.

Behaviors (methods) can only be called on instances (not the classes themselves), such as `mathNotes.addPage(..)` and `page.print()`.

The class mechanism allows packaging data to be organized together with their behaviors. The same program could have been built without any class definitions, but it would likely have been much less organize, harder to read and reason about, and more susceptible to bugs and subpar maintenance. 

## Class Inheritance 

Another aspect inherent to traditional "class-oriented" design, though a bit less commonly used in JS, is "inheritance" (and "polymorphism"). Consider: 

    class Publication {
        constructor(title, author, pubDate) {
            this.title = title; 
            this.author = author; 
            this.pubDate = pubDate;
        }
    }

        print() {
            console.log(`
                Title: ${ this.title }
                By: ${ this.author }
                ${ this.pubDate }
            `);
        }
    }

This `Publication` class defines a set of common behavior that any publication might need. 

Let's consider more specific types of publication, like `Book` and `BlogPost`:

    class Book extends Publication {
        constructor(bookDetails){
            super(
                bookDetails.title,
                bookDetails.author, 
                bookDetails.pulishedOn
            );
            this.publisher = bookDetails.publisher;
            this.ISBN = bookDetails.ISBN;
        }

        print() {
            super.print();
            console.log(`
                Publisher: ${ this.publisher }
                ISBN: ${ this.ISBN}
                `);
        }
    }

    class BlogPost extends Publication {
        constructor(title, author, pubDate, URL) {
            super(title, author, pubDate);
            this.URL = URL;
        }

        print() {
            super.print();
            console.log(this.URL);
        }
    }


Both `Book` and `BlogPost` use the `extends` clause to `extend` the general definition of `Publication` to include additional behavior: The `super(..)` call in each constructor delegates to the parent `Publication` class's constructor for its initialization work, and then they do more specific things according to their respective publication type. 

Consider using these child classes: 

    var YDKJS = new Book({
        title: "You Don't Know JS", 
        author: "Kyle Simpson",
        publishedOn: "June 2014",
        publisher: "O'Reilly",
        ISBN: "123456-789"
    });

    YDKJS.print(); 
    //Title: You Don't Know JS 
    //By: Kyle Simpson
    //June: 2014
    //Publisher: O'Reilly
    //ISBN: 123456-789

    var forAgainstLet = new BlogPost(
        "For and against let",
        "Kyle Simpson",
        "October 27, 2014",
        "https://davidwalsh.name/for-and-against-let"
    );

    forAgainstLet.print();
    // Title: For and against let
    // By: Kyle Simpson
    // October 27, 2014
    // https://davidwalsh.name/for-and-against-let

Notice that both child class instances have a `print()` method, which was an override of the inherited `print()` method from the parent `Publication` class. Each of those overriden child class `print()` methods call `super.print()` to invoke the inherited version of the `print()` method. 

The fact that both the inherited and overridden methods can have the same name and co-exist is called polymorphism. 

## Modules 

The module pattern has essentially the same goal as the class pattern, which is to group data and behavior together into logical units. Just like classes modules can "include" or "access" the data and bahaviors of other modules, for cooperation sake. 

## Classic Modules 

The key hallmarks of a classic module are an outer function (that runs at least once), which returns an "instace" of the module with one or more functions exposed that can operate on the module instance's internal(hidden) data.

Because a module of this form is just a function, and calling it produces an "instace" of the module, another description for these functions is "module factories".

Consider the classic module form of the earlier `Publication`, `Book`, and `BlogPost` classes: 

    function Publication(tile, author, pubDate) {
        var publicAPI = {
            print() {
                console.log(`
                    Title: ${ title }
                    By: ${ author }
                    ${ pubDate }
                `);
            }
        };

        return publicAPI;
    }

    function Book(bookDetails){
        var pub = Publication(
            bookDetails.title,
            bookDetails.author, 
            bookDetails.publishedOn
        );

        var publicAPI = {
            print() {
                pub.print();
                console.log(`
                    Publisher: ${ bookDetails.publisher }
                    ISBN: ${ bookDetails.ISBN }
                `);
            }
        };

        return publicAPI;
    }

    function BlogPost(title, author, pubDate, URL) {
        var pub = Publication(title, author, pubDate);

        var publicAPI = {
            print() {
                pub.print();
                console.log(URL);
            }
        };

        return publicAPI;
    }

Comparing these forms to the `class` forms, there are more similarities than differences. 

The `class`form stores methods and data aon an object instance, which must be accesses with the `this.` prefix. With modules, the methods and data are accessed as identifier variables in scope, without any `this.` prefix. 

With the module factory function, you explicitly create and return an object with any publicly exposed methods, and any data or other unreferenced methods remain private inside the factory function. 

There are other variations to this factory function that are quite common across JS. Still, all of these forms rely on the basic principles. 

Consider also the usage (aka, "instantiation") of these module factory functions: 

    var YDKJS = Book({
        title: "You Don't Know JS",
        author: "Kyle Simpson",
        publishedOn: "June 2014",
        publisher: "O'Reilly",
        ISBN: "123456-789"
    });

    YDKJS.print();
    // Title: You Don't Know JS
    // By: Kyle Simpson
    // June 2014
    // Publisher: O'Reilly
    // ISBN: 123456-789

    var forAgainstLet = BlogPost(
        "For and against let",
        "Kyle Simpson", 
        "October 27, 2014",
        "https://davidwalsh.name/for-and-against-let"
    );

    forAgainstLet.pring();
    // Title: For and against let
    // By: Kyle Simpson
    // October 27, 2014
    // https://davidwalsh.name/for-and-against-let

The only difference here is the lack of using `new`, calling the module factories as normal functions. 

## ES Modules 

ES modules (ESM), introduced to the JS language in ES6, are meant to serve much the same spirit and purpose as the existing classic modules just described.

The implementation approach does, however, differ significantly.

First, there's no wrapping function to define a module. The wrapping context is a file. ESMs are always file-based; one file, one module. 

Second, you don't interact with a module's "API" explicitly, but rather use the `export` keyword to add a variable or method to its public API definition. If something is defined in a module but not `exported`, then it stays hidden. 

Third, you don't "instantiate" an ES module, you just `import` it to use its single instance. ESMs are, in effect, "singletons", in that there's ony one instance ever created, at first `import` in your program, and all other `imports` just receive a reference to that same single instance. If your module needs to support multiple instantiations, you have to provide a classic module-style factory function on your ESM definition for that purpose. 

Consider the file `publication.js`, with a mix of both ESM and classic modules: 

    function printDetails(title, author, pubDate) {
        console.log(`
            Title: ${ title }
            By: ${ author }
            ${ pubDate }
        `);
    }

    export function create(title, author, pubDate) {
        var publicAPI = {
            print() {
                printDetails(title, author, pubDate);
            }
        };

        return publicAPI;
    }

To import and use this module, from another ES module like `blogpost.js`: 

    import { create as createPub } from "publication.js";

    function printDetails(pub, URL) { 
        pub.print();
        console.log(URL);
    }

    export function create(title, author, pubDate, URL) {
        var pub = createPub(title, author, pubDate);

        var publicAPI = {
            print() {
                printDetails(pub, URL);
            }
        };

        return publicAPI;
    }

And finally, to use this module, we import into another ES module like `main.js`:

    import { create as newBlogPost } from "blogpost.js";

    var forAgainstLet = newBlogPost(
        "For and against let",
        "Kyle Simpson",
        "October 27, 2014",
        "https://davidwalsh.name/for-and-against-let"
    );

    forAgainstLet.print();
    // Title: For and against let
    // By: Kyle Simpson
    // October 27, 2014
    // https://davidwalsh.name/for-and-against-let

As shown, ES modules can use classic modules internally if they need to support multiple-instantiation. 

If your module only needs a single instancem you can skip the extra layers of complexity: `export` its public methods directly. 