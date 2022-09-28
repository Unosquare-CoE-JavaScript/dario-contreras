# Chapter 6: Limiting Scope Exposure

We're going to look at how and why we should be using different leves of scope (function and blacks) to organize our program's variables, specifically to reduce scope over-exposure. 

##  Least Exposure

Software engineering articulates a fundamental discipline, typically applied to software security, called "The Principle of Least Privilege" (POLP). And a variation of this principle that applies to our current discussion is labeled as "Least Exposure" (POLE).

POLP expresses a defensive posture to software architecture: components of the system should be designed to function with least privilege, least access, least exposure. If each piece is connected with minimum-necessary capabilities, the overall system is stronger from a security standpoint, because a compromise or failure of one piece has a minimized impact on the rest of the system. 

If POLP focuses on system-level component design, the POLE exposure variant focuses on a lower level; we'll apply it to how scopes interact with each other. 

In following POLE, what do we want to minimize the exposure of? Simply: the variables registered in each scope.

When variables used by one part of the program are exposed to another part of the program, via scope, there are three main hazards that often arise: 

- **Naming Collisions**: if you use a common and useful variable/function name in two different parts of the program, but the identifier comes from one shared scope (like the global scope), then name collision occurs, and it's very likely that bugs will occur as one part uses the variable/function in a way the other part doesn't expect. 
- **Unexpected Behavior**: if you expose variables/functions whose usage is otherwise private to a piece of the program, it allows other developers to use them in ways you didn't intend, which can violate expected behavior and cause bugs. Also, exposure of private details invites those with mal-intent to try to work around limitations you have imposed, to do things with your part of the software that shoudn't be allowed. 
- **Unintended Dependency**: if you expose variables/functions unnecessarily, it invites other developers to use and depend on those otherwise private pieces. While that doesn't break your program today, it creates a refactoring hazard in the future, because now you cannot as easily refactor that variable or function without potentially breaking other parts of the software that you don't control. 

POLE, as applied to variable/function scoping, essentially says, default to exposing the bare minimum necessary, keeping everything else as private as possible. Declare variables in as small and deeply nested of scopes as possible, rather than placing everything in the global scope. 

If you design your software accordingly, you have a much greater chance of avoiding (or at least minimizing) these three hazards. 

Consider:

    function diff(x,y) {
        if (x > y) {
            let tmp = x;
            x = y;
            y = tmp; 
        }

        return y - x; 
    }    

    diff(3,7);      // 4
    diff(7,5);      // 2

In this simple example, it doesn't seem to matter whether `tmp` is inside the `if` block or whether it belongs at the function level. However, following the POLE principle, `tmp` should be as hidden in scope as possible. So we block scope `tmp` using `let` to the `if` block.

##  Hiding in Plain (Function) Scope

What about hiding `var` or `function` declarations in scopes? That can easily be done by wrapping a `function` scope around a declaration. 

Let's consider an example where `function` scoping can be useful. 

    var cache = {};

    function factorial(x) {
        if (x < 2) return 1;
        if(!(x in cache)) {
            cache[x] = x * factorial(x - 1); 
        }
        return cache[x];
    }

    factorial(6);
    // 720

    cache;

    // {
    //      "2": 2,
    //      "3": 6,
    //      "4": 24,
    //      "5": 120,
    //      "6": 720,
    // }

    factorial(7);
    // 5040

We're storing all the computed factorials in `cache` so that across multiple calls to `factorial(..)`, the previous computations remain. But the `cache` variable is pretty obviously a private detail of how `factorial(..)` works, not something that should be exposed in an outer scope. 

However, fixing this over-exposure issue is not as simple as hiding the `cache` variable inside `factorial(..)`, as it might seem. Since we need `cache` tu survive multiple calls, it mus be located in a scope outside that function. What can we do? 

Define another middle scope(between the outer/global scope and the inside of `factorial(..)`) for `cache` to be located:

    // outer/global scope

    function hideTheCache() {
        // "middle scope", where we hide `cache`
        var cache = {};

        return factorial; 

        // ************

        function factorial(x) {
            // inner scope
            if (x < 2) return 1; 
            if (!(x in cache)) {
                cache[x] = x * factorial(x - 1);
            }
            return cache[x];
        }
    }

    var factorial = hideTheCache(); 

    factorial(6);
    // 720

    factorial(7);
    // 5040

The `hideTheCache()` function serves no other purpose than to create a scope for `cache` to persist in across multiple calls to `factorial(..)`.

In this example, it's going to be tedious to define a `hideTheCache(..)` function scope each time such a need for variable/function hiding occurs, especially since we'll likely want to avoid name collisions with this function by giving each occurrence a unique name, so perhaps, a better solution is to use a function expression:

    var factorial = (function hideTheCache() {
        var cache = {};
        
        function factorial(x) {
            if (x < 2) return 1; 
            if (!(x in cache)) {
                cache[x] = x * factorial(x - 1);
            }
            return cache[x];
        }

        return factorial;
    })();

    factorial(6);
    // 720

    factorial(7);
    // 5040

Since `hideTheCache(..)` is defined as a `function` expression instead of a `function` declaration, its name is in its own scope - the same scope as `cache` - rather than in the outer/global scope. 

That means we can name every single occurrence of such a function expression the exact same name, and never have any collision. More appropriately, we can name each occurrence semantically based on whatever it is we're trying to hide, and not worry that whatever name we choose is going to collide qirh any other `function` expression scope in the program. 

## Invoking Function Expressions Immediately

Notice that we sorrounded the entire previous `function` expresion in a set of ( .. ), and then on the end, we added that second () parentheses set; that's actually calling the `function` expression we just defined. 

So, in other words, we're defining a `function` expression that's then immediately invoked. This common pattern has a name: Immediately Invoke Function Expression (IIFE).

An IIFE is useful when we want to create a scope to hide variables/functions . Since it's an expression, it can be used in any place in a JS program where an expression is allowed. An IIFE can be named, as with `hideTheCache()`, or unnamed/anonymous. And it can be standalone or, as before, part of another statement. 

Here's an example of a standalone IIFE: 

    // outer scope

    (function(){
        // inner hidden scope
    })();

    // more outer scope

##  Function Boundaries 

Beware that using an IIFE to define a scope can have some unintended consequences, depending on the code around it. Because an IIFE is a full function, the function boundary alters the behavior of certain statement/constructs. 

For example, a `return` statement in some piece of code would change its meaning if an IIFE is wrapped around it, because now the `return` would refer to the IIFE's function. Non-arrow function IIFEs also change the binding of a `this` keyword. And statements like `break` and `continue` won't operate across an IIFE function boundary to control an outer loop or block. 

In other words, if the code you need to wrap a scope around has `return`, `this`, `break`, or `continue` in it, an IIFE is probably not the best approach. In that case, you might look to create the scope with a block instead of a function.

##  Scoping with Blocks

A block only becomes a scope if necessary, to contain its block-scoped declarations(i.e., `let` or `const`). Consider: 

    {
        // not necessarily a scope (yet)

        // ..

        /
        / now we know the block needs to be a scope
        let thisIsNowAScope = true; 

        for (let i = 0; i < 5; i++) {
            // this is also a scope, activated each
            // iteration
            if (i % 2 == 0) {
                // this is just a block, not a scope
                console.log(i);
            }
        }
    }
    // 0 2 4

Not all `{ .. }` curly-brace pairs create blocks (and thus are eligible to become scopes):

- Object literals use `{ .. }` curly-brace pairs to delimit their key-value lists, but such object values are not scopes.
-  `class` uses `{ .. }` curly-braces around its body definition, but this is not a block or scope. 
- A `function` uses `{ .. }` around its body, but this is not technically a block - it's a single statement for the function body. It is, however, a function scope. 
- The `{ .. }` curly-brace pair on a `switch` statement does not define a block/scope. 

An explicit block scope can be useful even inside of another block (whether the outer block is a scope or not).

For example: 

    if (somethingHappened) {
        // this is a block, but not a scope

        {
            // this is both a block and an
            // explicit scope
            let msg = somethingHappened.message();
            notifyOthers(msg);
        }
        // .. 

        recoverFromSomething();
    }

Here, the `{ .. }` curly-brace pair inside the `if` statement is an even smaller inner explicit block scope for `msg`, since that variable is not needed for the entire `if` block.

If you find yourself placing a `let` declaration in the middle of a scope, first think about the TDZ alert. If this `let` declaration isn't needed in the first half of that block, you should use an inner explicit block scope to further narrow its exposure. 

Another example: 

    function getNextMonthStart(dateStr) {
        var nextMonth, year;

        {
            let curMonth;
            [, year, curMonth ] = dateStr.match(
                /(\d{4})-(\d{2})-\d{2}/
            ) || [];
            nextMonth = (Number(curMonth) % 12) + 1;
        }

        if (nextMonth == 1) {
            year++;
        }

        return `${ year }-${
            String(nextMonth).padStart(2, "0")
        }-01`;
    }

    getNextMonthStart("2019-12-25");            // 2020-01-01

Why put `curMonth` in an eplicit block scope instead of just alongside `nextMonth` and `year` in the top-level function scope? Because `curMonth` is only needed for those first two statements; at the function scope level it's over-exposed. 

##  `var` and `let`

`var` better communicates function-scoped than `let` does, and `let` both communicates and achieves block-scoping where `var` is insufficient.

##  Where to `let`

The advice to reserve `var` for (mostly) only top-level function scope means that most other declarations should use `let`.But you may still be wondering how to decide where each delaration in your program belongs? 

The way to decide is not based on which keyword you want to use. The way to decide is to ask, "What is the most minimal scope exposure that's sufficient for this variable?"

Once that is answered, you'll know if a variable belongs in a block scope or the function scope. If you decide initially that a variable should be block-scoped, and later realize it need to be elevated to be function-scoped, then that dictates a change not only in the location of that variale's declaration, but also the declarator keyword used. 

##  What's the Catch?

Since the introduction  of `try`..`catch` back in ES3, the `catch` clause has used an additional block-scoping declaration capability. 

    try {
        doesntExist();
    }
    catch (err) {
        console.log(err); 
        // RerenceError: 'doesntExist' is not defined
        // ^^^^ message printed from the caught exception

        let onlyHere = true; 
        var outerVariable = true;
    }

    console.log(outerVariable);         // true

    console.log(err);   
    // ReferenceError: 'err' is not defined 
    // ^^^^ this is another thrown ()

The `err` variable declared by the `catch` clause is block-scoped to that block. This `catch` clause can hold other block-scoped declarations via `let`. But a `var` declaration inside this block still attaches to the outer function/global scope. 

ES2019 (recently, at the time of writing) changed catch clauses so their declaration is optional; if the declaration is omitted, the catch block is no longer (by default) a scope; it’s still a block, though! So if you need to react to the condition that an exception occurred (so you can gracefully recover), but you don’t care about the error value itself, you can omit the catch declaration:

    try {
        doOptionOne();
    }
    catch {     //catch -declaration omitted
        doOptionTwoInstead();
    }

##  Function Declarations in Blocks (FiB)

Consider: 

    if (false) {
        function ask() {
            console.log("Does this run?");
        }
    }

    ask();

What do you expect for this program to do? Three reasonable outcomes: 

1. Tha ask() call might fail with a `ReferenceError` exception, because the `ask` identifier is block-scoped to the `if` block scope and thus isn't available in the outer/global scope. 
2. The `ask()` call might fail with a `TypeError` exception, because the `ask` identifier exists, but it's `undefined` and thus not a callable function. 
3. The `ask()` call might run correctly, printing out the "Does it run?" message. 

Depending on which JS environment you try that code snippet in, you may get different results. This is one of those fre crazy areas where existing legacy behavior betrays a predictable outcome. 

The JS specification says that `function` declarations inside of blocks are block-scoped, so the answer should be (1). However, most browser-based JS engines will behave as (2), meaning the identifier is scoped outside the `if` block but the function value is not automatically initialized, so it remains `undefined`.

Why are browser JS engines allowed to behave contrary to the specification? Because these engines already had certain be-haviors around FiB before ES6 introduced block scoping, and there was concern that changing to adhere to the specification might break some existing website JS code. 

One of the most common use cases for placing a `function` declaration in a block is to conditionally define a funcdtion one way or another (like with an `if.. else` statement) depending on some environment state. For example: 

    if (typeof Array.isArray != "undefined") {
        function isArray(a) {
            return Array.isArray(a);
        }
    } else {
        function isArray(a) {
            return Object.prototype.soString.call(a) == "[object Array]";
        }
    }

There are some vagaries of FiB, but the best solution to avoid them, is to simply not use FiB entirely. In other words, never place a `function` declaration directly inside any block. Always place `function` declarations anywhere in the top-level scope of a function (or in the global scope).

FiB is not worth it, and should be avoided. 

##  Blocked Over

The point of lexical scoping rules in a programming language is so we can appropriately organize our program's variables, both for operation as well as semantic code communication purposes. 

One of the most important organizational techniques is to ensure that no variable is over-exposed to unnecessary scopes (POLE).
