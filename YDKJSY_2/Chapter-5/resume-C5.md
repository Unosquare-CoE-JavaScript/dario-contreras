# Chapter 5: The (Not So) Secret Lifecycle of Varibles. 

Knowing which scope a variable comes from is only part of the story. If a variable declaration appears past the first statement of a scope, how will any references to that identifier before the declaration behave? What happens if you try to declare the same variable twice in a scope? 

JS's particular flavor of lexical scope is rich with nuance in how and when variables come into existence and become available to the program. 

## When Can I Use a Variable? 

At what point does a variable become available to use within its scope? 

Consider: 

    greeting(); 
    // Hello!

    function greeting() {
        console.log("Hello!");
    }

This code works fine. Did you ever wonder how or why it works? Why can you access the identifier `greeting` from line 1, even though the `greeting()` function declaration doesn't occur until line 4? 

The term most commonly used for a variable being visible from the beginning of its enclosing scope, ,even though its declaration may appear further down in the scope, is called **hoisting**.

But, how does the variable `greeting` have any value assigned to it, from the moment the scope starts running? The answer is a special characteristic of formal `function` declaration's, called function hoisting. 

When a `function` declaration's name identifier is registered at the tope of its scope, it's additionally auto-initialized to that function's reference. That's why the function van be called throughout the entire scope. 

One key detail is that both function hoisting and `var`-flavored variable hoisting attach their name identifiers to the nearest enclosing function scope (or, if none, the global scope), not a block scope. 

##  Hoisting: Declaration vs Expression 

Function hoisting only applies to formal `function` declarations, not to `function` expression assignments. 

Consider: 

    greeting(); 
    // TypeError 

    var greeting = function greeting() {
        console.log("Hello!");
    };

Line 1 throws an error. But the kind of error thrown is very important to notice. A `TypeError` means we're trying to do something with a value that is not allowed. Depending on the JS environment, the error message would say something like, "undefined is not a function", or more helpfully, "greeting is not a function".

Notice that the error is not a `ReferenceError`. JS isn't telling us that it couldn't find `greeting` as an identifier in the scope. It's telling us that `greeting` was found but doesn't hold a function reference at that moment. Only functions can be invoked, so attempting to invoke some non-function value results in an error. 

But, what does `greeting` hold, if not the function reference? 

In addition to being hoisted, variables declared with `var` are also automatically initialized to `undefined` at the beginning of their scope. So on that first line, `greeting` exists, but it holds only the default `undefined` value. It's not until line 4 that `greeting` gets assigned the function reference. 

Pay close attention to the distinction here. A `function` declaration is hoisted and initialized to its function value(again,called function hoisting). A `var` variable is also hoisted, and then auto-initialized to `undefined`. Any subsequent function expression assignments to that variable don’t happen until that assignment is processed during runtime execution.

In both cases, the name of the identifier is hoisted. But the function reference association isn't handled at initialization time unless the identifier was created in a formal function declaration. 

## Variable Hoisting 

Let's look at another example of variable hoisting: 

    greeting = "Hello!";
    console.log(greeting);
    // Hello!

    var greeting = "Howdy!";

Though `greeting` isn't declared until line 5, it's available to be assigned to as early as line 1. Why? 

- The identifier is hoisted
- It's automatically initialized to the value `undefined` from the top of the scope

## Re-declaration? 

What do you think happens when a variable is declared more than once in the same scope? Consider: 

    var studentName = "Frank";
    console.log(studentName);
    // Frank

    var studentName; 
    console.log(studentName);       // Frank

Since hoisting is actually about registering a variable at the beginning of a scope, there's nothing to be done in the middle of the scope where the original program actually had the second `var studentName` statement. It's just a no-op(eration), a pointless statement. 

Also, note that `var studentName` doesn't mean `var studentName = undefined;`, as most could assume. 

Consider: 

    var studentNmae = "Frank";
    console.log(studentName);           // Frank

    var studentName; 
    console.log(studentName);           // Frank <--- still! 

    // let's add the initialization explicitly 
    var studentName = undefined; 
    console.log(studentName);           // undefined <--- see!?

A repeated `var` declaration of the same identifier name in a scope is effectively a do-nothing operation.

What about repeating a declaration within a scope using `let` or `const`?

    let studentName = "Frank";

    console.log(studentName);

    let studentName = "Suzy";

This program will not execute, but instead immediately throw a `SyntaxError`. Depending on your JS environment, the error message will indicate something like: "studentName has already been declared." In other words, this is a case where attempted "re-declaration" is explicitly not allowed. 

It's not just that two declarations involving `let` will throw this error. If either declaration uses `let`, the other can be either `let` or `var`, and the error will still occur.

The reason for the error is not technical per se, as `var` "re-declaration" has always been allowed; the same allowance could have been made for `let`.

This is more of a social engineering issue. "Re-declaration" of variables is seen by some as a bad habit that can lead to program bugs. So when ES6 introduced `let`, they decided to prevent "re-declaration" with an error. 

##  Constants? 

The `const` keyword is more constrained than `let`. Like `let`, `const` cannot be repeated with the same identifier in the same scope.   

The `const` keyword requires a variable to be initialized, so omitting an assignment from the declaration results in a `SyntaxError`: 

    const empty;        // SyntaxError

`const` declarations create variables that cannot be re-assigned: 

    const studentName = "Frank";
    console.log(studentName);
    // Frank

    studentName = "Suzy";           // TypeError

The `studentName` variable cannot be re-assigned because it's declared with a `const`.

##  Loops 

Consider: 

    var keepGoing = true; 

    while (keepGoing) {
        let value = Math.random();
        if (value > 0.5) {
            keepGoing = false;
        }
    }

Is `value` being "re-declared" repeatedly in this program? Will we get errors thrown? No.  

All the rules of scope are applied per scope instance. In other words, each time a scope is entered during execution, everything resets. 

Each loop iteration is its own new scope instance, and within each scope instance, `value` is only being declared once. So there's no attempted "re-declaration", and thus no eror. 

Now consider: 

    var keepGoing = true; 

    while (keepGoing) {
        var value = Math.random();
        if (value > 0.5) {
            keepGoing = false;
        }
    }

Is `value` being "re-declared" here, especially since we know `var` allows it? No. `var` is not treated as a blockscoping declaration, it attaches itself to the global scope. So there's just one `value` variable, in the same scope as `keepGoing`. No re-declaration here either. 

What about "re-declaration" with other loop forms, like `for`-loops? 

    for (let i = 0; i < 3; i++) {
        let value = i * 10;
        console.log(`${ i }: ${ value }`);
    }
    // 0: 0
    // 1: 10
    // 2: 20 

It should be clear that there's only one `value` declared per scope instance. But what about `i`? Is it being "re-declared"? 

To answer that, consider what scope `i` is in, which is the scope of the `for`-loop body. just like `value` is. You could sorta think about that loop in this more verbose equivalent form: 

    {
        // a fictional variable for illustration
        let $$i = 0;

        for ( /* nothing */; $$i < 3; $$i++>) {
            // here's our actual loop `i`!
            let i = $$i;

            let value = i * 10;
            console.log(`${ i }: ${ value }`);
        }
        // 0: 0
        // 1: 10
        // 2: 20
    }

Now it should be clear: the `i` and `value` variables are both declared exactly once per scope instance. No "re-declaration" here. 

What about other `for`-loops forms?

    for (let index in students) {
        // this is fine
    }

    for (let student of students) {
        // so is this
    }

Same thing with `for`..`in` and `for`..`of` loops: the declared variable is treated as inside the loop body, and thus is handled per iteration (aka, per scope instance). No "re-declaration".

`const` inside loops works just like the `let` variant, being run exactly once within each loop iteratino, so it's safe from "re-declaration" troubles. However, things get more complicated when we talk about general `for`-loops.

    for (const i = 0; i < 3; i++) {
        // oops, this is going to fail with 
        // a Type Error after the first iteration
    }

So what's wrong here? Let's mentally expand that loop like we did earlier: 

    {
        // a fictional variable for illustration
        const $$i = 0; 

        for ( ; $$i < 3; $$i++) { 
            // here's our actual loop `i`!
            const i = $$i; 
            // .. 
        }
    }

The problem is the conceptual `$$i` that must be incremented each time with the `$$i++` expression. That's **re-assignment**, which isn't allowed for constants. 

`const` can't be used with the classic `for`-loop because of the requiered re-assignment. 

## Uninitialized Variables (aka, TDZ)

Consider: 

    console.log(studentName);
    // ReferenceError

    let studentName = "Suzy";

For `let`/`const`, the only way to initialize a variable is with an assignment attached to a declaration statement.  

    let studentName = "Suzy";
    console.log(studentName);   // Suzy

Alternatively:

    // ..

    let studentName;
    // or:
    // let studentName = undefined; 

    // ..

    studentName = "Suzy";

    console.log(studentName);
    // Suzy

The term coined by TC39 to refer to this period of time from the entering of a scope to where the auto-initialization of the variable occurs is: Temporal Dead Zone (TDZ).

The TDZ is the time window where a variable exists but is still uninitialized, and therefore cannot be accessed in any way. Only the execution of the instructions left by Compiler at the point of the original declaration can do that initialization. After that moment, the TDZ is done, and the variable is free to be used for the rest of the scope. 

A `var` also technically has a TDZ, but it's zero in length and thus unobservable to our programs, only `let` and `const` have an observable TDZ.

There's a common misconception that TDZ means `let` and `const` do not hoist. This is an inaccurate, or at least slightly misleading claim. They do hoist. 

The difference is that `let`/`const` declarations do not automatically initialize at the beginning of the scope, the way `var` does. 

Let's prove that `let` and `const` do hoist: 

    var studentName = "Kyle";

    {
        console.log(studentName);
        // ???

        // ..

        let studentName = "Suzy",

        console.log(studentName);
        // Suzy
    }

The first `console.log(..)` throws a TDZ error, because the inner scope's `studnetName` was hoisted (auto-registered at the top of the scope). What didn't happen uet was the auto-initialization of that inner `studentName`; it's still uninitialized at that moment, hence the TDZ violation.

How can you avoid TDZ errors? 

Always put your `let` and `const` declarations at the top of any scope. Shrink the TDZ window to zero length, and then it'll be moot. 
