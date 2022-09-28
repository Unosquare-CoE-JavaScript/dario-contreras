# Chapter 4: Around the Global Scope 

The global scope of a JS program is a rich topic, with much more utility and nuance than you would likely assume.  

Fully understanding the global scope is critical in your mastery of using lexical scope to structure your programs. 

##  Why Global Scope?

How exactly do all the separate files in JS get stitched together in a single runtime context by the JS engine? 

In respect to browser-executed applications, there are three main ways. 

First, if you're directly using ES modules, these files are loaded individually by the JS environment. Each module then `imports` references to whichever other modules it needs to access. The separate module files cooperate with each other exclusively through these shared impots, without needing any shared outer scope. 

Second, if you're using a bundler in your build process, all the files are typically concatenated together before delivery to the browser and JS engine, which then only processes one big file. Even with all the pieces of the application co-located in a single file, some mechanism is necessary for each piece to register a name to be referred to by other pieces, as well some facility for that access to occur. 

In some build setups, the entire contents of the file are wrapped in a single enclosing scope, such as a wrapper function, universal module, etc. Each piece can register itself for access from other pieces by way of local variables in that shared scope. 

For example: 

    (function wrappingOuterScope(){
        var moduleOne = (function one(){
            //..
        })();

        var moduleTwo = (function two(){
            // ..

            function callModuleOne(){
                moduleOne.someMethod();
            }
            // ..
        })();
    })();

The `moduleOne` and `moduleTwo` local variables inside the `wrappingOuterScope()` function scope are declared so that these modules can access each other for their cooperation. 

While the scope of `wrappingOuterScope()`is a function and not the full environment global scope, it does act as a sort of “application-wide scope,” a bucket where all the top-level identifiers can be stored, though not in the real global scope. It’s kind of like a stand-in for the global scope in that respect.

The third and final way, whether a bundler tool is used for an application, or whether the files are simply loaded in the browser individually, if there is no single sorrounding scope encompassing all these pieces, the global scope is the only way for them to cooperate with each other:  

    var moduleOne = (function one(){
        // ..
    })(); 
    
    var moduleTwo = (function two()){
        // ..

        function callModuleOne() {
            moduleOne.someMethod();
        }

        // ..
    })();

Here, since there is no sorrounding function scope, these `moduleOne` and `moduleTwo` declarations are simply dropped into the global scope. 

If these files are loaded separately as normal standalone .js files in a browser environment, each top-level variable declaration will end up as a global variable, since the global scope is the only shared resource between these two separate files.  

To account for where an applicatoin's code resides during runtime, and how each piece is able to access the other pieces to cooperate, the global scope is also where:

- JS exposes its built-ins: 
    - primitives: `undefined`, `null`, `Infinity`, `NaN`
    - natives: `Date()`, `Object()`, `String()`, etc.
    - global functions: `eval()`, `parseInt()`, etc.
    - friends of JS: `Intl`, `WebAssembly`
- The encironment hosting the JS engine exposes its own built-ins: 
    - `console` (and its methods)
    - the `DOM` (`window`, `document`, etc)
    - web platform APIs:  `navigator`, `history`, geolocation, WebRTC, etc. 

These are just some of the many globals a JS program will interact with. 

##  Where Exactly is this Global Scope?

Different JS environments handle the scopes of your programs, especially the global scope, differently. 

##  Browser "Window"

With respect to treatment of the global scope, the most pure environment JS can be run is as a standalone .js file loaded in a web page environment in a browser.  

Consider this .js file: 

    var studentName = "Kyle";

    function hello() {
        console.log(`Hello, ${ studentName }!`);
    }

    hello(); 
    // Hello, Kyle!

This code may be loaded in a web page environment using an inline `<script>` tag, a `<script src=..>` script tag in the markup, or even a dynamically created `<script> DOM` element. In all three cases, the `studentName` and `hello` identifiers are declared in the global scope. 

That means that if you access the global object, you'll find properties of those same names there: 

    var studentName = "Kyle";

    function hello() {
        console.log(`Hello, ${ window.studentName }!`);
    }

    window.hello();
    // Hello, Kyle!

That's the default behavior one would expect from a reading of the JS specification: the outer scope is the global scope and `studentName` is legitimately created as global variable. 

However, this will not be true for all the JS environments we can encounter. 

## Global Shadowing Globals 

An unusual consequence of the difference between a global variable and a global property of the same name is that, within just the global scope itself, a global object property can be shadowed by a global variable: 

    window.something = 42; 

    let something = "Kyle";

    console.log(something);
    // Kyle

    console.log(window.something);
    // 42

It's a bad idea to create a divergence between the global object and the global scope. A simple way to avoid this problem with global declarations is to always use `var` for globals. Reserve `let` and `const` for block scopes. 

##  DOM Globals

One surprising behavior in the global scope you may encounter with browser-based JS applications: a DOM element with an `id` attribute automatically creates a global variable that references it. 

Consider: 

    <ul id="my-todo-list">
        <li id="first">Write a book</li>
        ..
    </ul>

And the JS for that page could include: 

    first; 
    // <li id="first">..</li>

    window["my-todo-list"];
    // <ul id="my-todo-list">..</ul>

If the `id` value is a valid lexical name (like `first`), the lexical variable is created. If not, the only way to access that global is through the global object (`window[..]`).

The auto-registration of all `id`-bearing DOM elements as global variables is an ald legacy browser behabior that nevertheless must remain because so many ald sites still rely on it. 


##  What's in a (Window) Name? 

Another global scope oddity in browser-based JS: 

    var name = 42; 

    console.log(name, typeof name);
    // "42" string 

`window.name` is a pre-defined "global" in a browser context; it's a property on the global object, so it seems like a normal global variable. It's anything but normal. 

We used `var` for our declaration, which does not shadow the pre-defined `name` global property. 

With the exception of some rare corner cases like DOM element ID's and `window.name`, JS running as a standalone file in a browser page has some of the most pure global scope behavior we will encounter. 

##  Web Workers

Web Workers are a web platform extension on top of browser-JS behavior, which allows a JS file to run in a completely separate thread (operating system wise) from the thread that's running the main JS program. 

Since these Web Worker programs run on a separate thread, they're restricted in their communications with the main application thread, to avoid/limit race conditions and other complications. Somo web APIs are available to the worker though, such as `navigator`.

A Web Worker is trated as a wholly separate program, so it does not share the global scope with the main JS program. However, the browser's JS engine is still running the code, so we can expect similar purity of its global scope behavior. Since there is no DOM access, the `window` alias for the global scope doesn't exist. 

In a Web Worker, the global object reference is typically made using `self`:

    var studentName = "Kyle";
    let studentID = 42;

    function hello() {
        console.log(`Hello, ${ self.studentName }!`)
    }

    self.hello():
    // Hello, Kyle! 

    self.studentID; 
    // undefined  

Just as with main JS programs, `var` and `function` declarations create mirrored properties on the global object (aka `self`), where other declarations (`let`, etc) do not. 

##  Developer Tools Console/REPL 

When using the developer tools console, certain error conditions applicable to a JS program may be relaxed and not displayed when the code is entered into a developer tool. 

Such observable differences may include: 

- The behavior of the global scope. 
- Hoisting 
- Block-scoping declarators (`let` / `const`) when used in the outermost scope. 

In a nutshell, Developer Tools, while optimized to beconvenient and useful for a variety of developer activities, arenotsuitable environments to determine or verify explicit andnuanced behaviors of an actual JS program context.

## ES Modules (ESM)

ES6 instroduced first-class support for the module pattern. One of the most obvious impacts of using ESM is how it changes the behavior of the observably top-level scope in a file. 

Recall this code. 

    var studentName = "Kyle";

    function hello() {
        console.log(`Hello, ${ studentName }!`);
    }

    hello();
    // Hello, Kyle!

    export hello; 

If that code is in a file that's loaded as an ES module, it will still run exactly the same. However, the observable effects, from the overall application perspective, will be different. 

Despite being declared at the top level of the (module) file, in the outermost obvious scope, `studentName` and `hello` are not global variables. Instead, they are module-wide, or if you prefer, "module-global".

However, global variables don't get created by declaring variablese in the top-level scope of a module. 

The module's top-level scope is descended from the global scope, almost as if the entire contents of the module were wrapped in a function. Thus, all variables that exist in the global scope are availble as lexical identifiers from inside the module's scope. 

ESM encourages a minimization of reliance on the global scope, where you import whateverm odules you may need for the current module to operate. As such, you less often see usage of the global scope or its global object. 

##  Node 

Every .js file in Node, including the main one, is treated as a module (ES module or CommonJS module). The practical effect is that the top level of your Node programs is never actually the global scope, the way it is when loading a non-module file in the browser. 

Node has recently added support for ES modules, but from its beggining, it has supported a module format referred to as "CommonJS", which looks like this: 

    var studentName = "Kyle";

    function hello(){
        console.log(`Hello, ${ studentName }!`);
    }

    hello();    
    // Hello, Kyle!

    module.exports.hello = hello; 

Before processing, Node effectively wraps such code in a function, so that the `var` and `function` declarations are contained in that wrapping function's scope, not treated as global variables. 

Node defines a number of "globals" like `require()`, but they are not actually identifiers in the global scope of every module. They're injected in the scope of every module. 

The only way to define an actual global variable in Node is to add properties to another of Node's automatically provided "globals", which is ironically called `global`. `global` is a reference to the real global scope object, somewhat like using `window` in a browser JS environment. 

Consider:

    global.studentName = "Kyle";

    function hello() {
        console.log(`Hello, ${ studentName }!`);
    }

    hello();
    //Hello, Kyle!

    module.exports.hello = hello; 

Here we add `studentName` as a property on the `global` object, and then in the `console.log(..)` statement we're able to access `studentName` as a normal global variable. 

Note, the identifier `global` is not defined by JS; it's specifically defined by Node. 

## Global This 

Reviewing the JS environments we've looked at so far, a program may or may not: 

- Declare a global variable in the top-level scope with `var` or `function` declarations-or `let`, `const`, and `class`.
- Also add global variables declarations as properties of the global scope object if `var` or `function` are used for the declaration. 
- Refer to the global scope object (for adding or retrieving global variables, as properties) with `window`, `self`, or `global`.

A trick for obtaining a reference to the global scope object looks like: 

    const theGlobalScopeObject = 
        (new Function("return this"))();
    
So, we have `window`, `self`, `global`, and this ugly `new Function(..)` trick.  

As of ES2020, JS has finally defined a standardized reference to the global scope object, called `globalThis`. So, subject to the recency of the JS engines your ocde runs in, you can use `globalThis` in place of any of those other approaches. 

We could even attempt to define a cross-environment polyfill that's sager across pre-`globalThis` JS environments, such as: 

    const theGlobalObject = 
        (typeof globalThis != "undefined") ? globalThis:
        (typeof global != "undefined") ? global:
        (typeof window != "undefined") ? window:
        (typeof self != "undefined") ? self:
        (new Function("return this"))();

That's certainly not ideal, but it works if you find yourself needing a reliable global scope reference. 

##  Globally Aware 

The global scope is present and relevant in every JS program, even though modern patterns for organizing code into modules de-emphasizes much of the reliance on storing identifiers in that namespace.

It's important that we have a solid grasp on the differences in how the global scope behave across different JS environments. 


