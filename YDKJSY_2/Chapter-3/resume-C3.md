# Chapter 3: The Scope Chain
 
##  "Lookup" Is (Mostly) Conceptual

The suggestion of a runtime lookup process works well for conceptual understanding, but it's not actually how things usually work in practice. 

The color of a marble's bucket (aka, meta information of what scope a variable originates from) is usually determined during the initial compilation processing. Because lexical scope is pretty much finalized at that point, a marble's color will not change based on anything that can happen later during runtime. 

Since the marble's color is know from compilation, and it's immutable, this information would likely be stored with each variable's entry in the AST; that information is then used explicitly by the executable instruction that constitute the program's runtime. 

In other words, the engine doesn't need to lookup through a bunch of scopes to figure out which scope bucket a variable comes from. That information is already known! Avoiding the need for a runtime lookup is a key optimization benefit of lexical scope. 

In what case would a marble color not be known during compilation? 

Consider a reference to a variable that isn't declared in any lexically available scopes in the current file. Another file (program) in the runtime may indeed declare that variable in the shared global scope. 

The ultimate determination of whether the variable was ever appropriately declared in some accessible bucket may need to be deferred to the runtime. 

Any reference to a variable that's initially undeclared is left as an uncolored marble during that file's compilation; this color cannot be determined until other relevant file(s) have been compiled and the application runtime commences. That deferred lookup will eventually resolve the color to whichever scope the variable is found in (likely the global scope).

However, this lookup would only be needed once per variable at most, since nothing else during runtime could later change that marble's color. 

## Shadowing 

Where having different lexical scope buckets starts to matter more is when you have two or more vaiables, each in different scopes, with the same lexical names. A single scope cannot have two or more variables with the same name; such multiple references would be assumed as just one variable.

So if you need to maintain two or more variables of the same name, you must use separate scopes. 

Consider: 

    var studentName = "Suzy";

    function printStudent(studentName) {
        studentName = studentName.toUpperCase();
        console.log(studentName);
    }

    printStudent("Frank");
    // FRANK

    printStudent(studentName);
    // SUZY 

    console.lgo(studentName);
    // Suzy

Considering the variable `studentName` and the conceptual notion of the "lookup", we asserted that it starts with the current scope and works its way outward/upward, stopping as soon as a matching variable is found. 

This is a key aspect of lexical scope behavior, called shadowing. The `studentName` variable inside of the `printStudent()` functionn is shadowing the shadowed global variable. 

When you choose to shadow a variable from an outer scope, one direct impact is that from that scope inward/downward it's now impossible for any marble to be colored as the shadowed variable. 

## Global Unshadowing Trick 

It is possible to access a global variable from a scope where that variable has been shadowed, but not through a typical lexical identifier reference.

In the global scope, `var` declarations and `function` declarations also expose themselves as properties on the global object. 

Consider this program, executed as an standalone .js file in a browser environment: 

    var studentName = "Suzy";  

    function printStudent(studentName) {
        console.log(studentName);
        console.log(window.studentName);
    }

    printStudent("Frank");
    // "Frank"
    // "Suzy"

The `window.studentName` expression is accessing the global variable `studentName` as a property on `window` (which we're pretending for now is synonymous with the global object). That's the only way to access a shadowed variable from inside a scope where the shadowing variable is present. 

The `window.studentName` is a mirror of the global `studentName` variable, not a separate snapshot copy. Changes to one are still seen from the other, in either direction. 

Other forms of global scope declarations do not create mirored global object properties: 

    var one = 1; 
    let notOne = 2; 
    const notTwo = 3; 
    class not Three = {}

    console.log(window.one);                // 1
    console.log(window.notOne);             // undefined
    console.log(window.notTwo);             // undefined
    console.log(window.notThree);           // undefined
    
Variables that exist in any other scope than the global scope are completely inaccessible from a scope where they've been shadowed:

    var special = 42; 

    function lookingFor(special) {
        // The identifier `special` (parameter) in this
        // scope is shadowed inside keepLooking(), and 
        // is thus inaccessible from that scope

        function keepLooking() {
            var special = 3.141592; 
            console.log(special);
            console.log(window.special);
        }        

        keepLooking();
    }

    lookingFor(112358132134);
    // 3.141592
    // 42

The global `special` is shadowed by the function parameter `special`. and the function `special` is itself shadowed by the `special` var inside the `keepLooking()` function. 

We can still access the global `special` using the indirect reference `window.special`. But there's no way for `keepLooking()` to access the `lookingFor() special` parameter that holds the number `112358132134`.

## Illegal Shadowing 

Not all combinations of declaration shadowing are allowed. `let` can shadow `var`, but `var` cannot shadow `let`: 

    function something() {
        var special = "JavaScript";

        {
            let special = 42;   // totally fine shadowing

            // ..
        }
    }   

    function another() {
        // ..
        {
            let special = "JavaScript";
            
            {
                var special = "JavaScript";
                // ^^^ Syntax Error

                // ..
            }
        }
    }

Notice in the `another()` function, the `inner var special` declaration is attempting to declare a function-wide `special`, which in and of itself is fine. 

The syntax error description in this case indicates that `special` has already been defined, but that error message is a little misleading. 

The real reason it's raised as a `SyntaxError` is because the `var` is basically trying to "cross the boundary" of the `let` declaration of the same name, which is not allowed. 

Consider: 

    function another() {
        // ..

        {
            let special = "JavaScript";

            ajax("https://some.url", function callback() {
                // totally fine shadowing
                var special = "JavaScript";

                // ..
            });
        }
    }

Summary: `let` in an inner scope can always shadow an outer scope's `var`. `var` in an inner scope can only shadow an outer scope's `let` if there is a function boundary in between. 

## Function Name Scope 

A function looks like this: 

    function askQuestion() {
        // ..
    }

Such a function declaration will create an identifier in the enclosing scope names `askedQuestion`.

Consider: 

    var askQuestion = function() {
        // ..
    };

The same is true for the variable `askQuestion` being created. But since it's a `function` expression, the function itself will not "hoist".

One major difference between `function` delcarations and `function` expressions is what happens to the name identifier of the function. 

Consider: 

    var askQuestion = function ofTheTeacher(){
        // ..
    };

We know `askQuestion` ends up in the outer scope. But what about `ofTheTeacher` identifier? For formal `function` declarations, the name identifier ends up in the outer/enclosing scope, so it may be reasonable to assume that's true here. However, `ofTheTeacher` is declared as an identifier inside the function itself: 

    var askQuestion = function ofTheTeacher() {
        console.log(ofTheTeacher);
    };

    askQuestion(); 
    // function ofTheTeacher()...

    console.log(ofTheTheacher);
    // RefernceError: ofTheTeacher is not defined

Not only is `ofTheTeacher()` declared inside the function rather than outside, but it's also defined as read-only: 

    var askQuestion = function ofTheTeacher() {
        "use strict";
        ofTheTeacher = 42;      // TypeError

        //..
    };

    askQuestion(); 
    // TypeError

Because we used strict-mode, the assignment failure is reported as a `TypeError`; in non-strict-mode, such an assgnment fails silently with no exception. 

What about when a `function` expression has no name identifier? 

    var askQuestion = function(){
        // ..
    };

A `function`expression with a name identifier is referred to as a "named function expression",  but one without a name identifier is referred to as an "anonymous function expression". Anonymoues function expressions clearly have no name identifier that affects either scope. 

## Arrow Functions 

ES6 added an additional `function` expression form to the language, called "arrow functions":

    var asQuestion = () => {
        // ..
    };

The `=>` arrow function doesn't require the word `function` to define it. Also, the `( .. )` around the parameter list is optional in some simple cases. Likewise, the `{ .. }` around the functino body is optional in some cases. And when the `{ .. }` are omitted, a return value is sent out without using a `return` keyword. 

Arrow functions are lexically anonymous, meaning they have no directly related identifier that references the function.  The assignment to `askQuestion` creates an inferred name of "askQuestion", but that's not the same as being non-anonymous: 

    var askQuestion = () => {
        // ..
    };

    askQuestion.name;           // askQuestion 

Arrow functions achieve their syntactic brevity at the expense of having to mentally juggle a bunch of variations for different forms/conditions. Just a few, for example: 

    () => 42; 

    id => id.toUpperCase();

    (id, name) => ({ id, name });

    (...args) => {
        return args[args.length - 1];
    };

Other than being anonymous, arrow functions have the same lexical scope rules as `function` functios do. An arrow function, with or without `{ .. }` around its body, still creates a separate, inner nested bucket of scope. Variable declarations inside this nested scope bucket behave the same as in a `function`scope. 

