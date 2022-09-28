## Values vs. References 

In many languages, the deveoper can choose between assigning/passing a value as the value itself, or as a referene to the value. In JS, however, this decision is entirely determined by the kind of value.  

If you assign/pass a value itself, the value is copied: 

    var myName = "Kyle";

    var yourName = myName; 

Here, the `yourName` variable has a separate copy of the "Kyle" string from the value that's stored in `myName`. That's because the value is a primitive, and primitive values are always assigned/passed as value copies. 

Here's how you can prove it: 

    var myName = "Kyle";

    var yourName = myName;

    myName = "Frank";

    console.log(myName);
    // Frank

    console.log(yourName)
    // Kyle

`yourName` wasn't affected by the re-assignment of `myName` to "Frank", that's because each variable hods its own copy of the value. 

By contrast, references are the idea that two or more variables are pointing at the same value, such that modifying this shared value would be reflected by an access via any of those references. In JS, only object values (arrays, objects, functions, etc.) are treated as references. 

Consider: 

    var myAdress = {
        street: "123 JS Blv",
        city: "Austin",
        state: "TX"
    };

    var yourAddress = myAddress; 

    // I've got to move to a new house!
    myAdress.strees = "456 TS Ave";

    console.log(yourAddress.street);
    // 456 TS Ave

Because the value assigned to `myAddress` is an object, it's held/assigned by reference, and thus the assignment to the `yourAddress` variable is a copy of the reference, not the object value itself. That's why the updated value asigned to the `myAddress.street` is reflected when we access `yourAddress.street`. `myAddress` and `yourAddress` have copies of the reference to the single shared object, so an update to one is an update to both.

## Function forms 

Consider:

    var awesomeFunction = function(coolThings) {
        // ..
        return amazingStuff;
    }

The function expression here is referred to as an anonymous function expression, since it has no name identifier between the `function` keyword and the `(..)` parameter list. This confuses many JS developers because as of ES6, JS performs a "name inference" on an anonymous function:

    awesomeFunction.name; 
    // "awesomeFunction"

The `name` property will reveal either its directly given name, or its inferred name in the case of an anonymous function expression. That value is used by developer tools when inspecting a function value or when reporting an error stack trace. 

Name inference only happens in limited cases such as when the function expression is assigned (with =). However, even if a name is inferred, it's still an anonymous function. Why? Because the inferred name is a medatada string value, not an available identifier to refer to the function. 

Consider: 

    var awesomeFunction = function someName(coolThings) {
        // ..
        return amazingStuff;
    }

    awesomeFunction.name; 
    // "someName"

This function expression is a named function expression, since the identifier `someName` is directly associated with the function expression at compile time; the associaion with the identifier `awesomeFunction` still doesn't happen until runtime at the time of that statement.  Those two identifiers don't have to match.

Notice also that the explicit function name, the identifier `someName`, takes precedence when assigning a name for the `name` property. 

Here are some more declaration forms: 

    // generator function declaration 
    function *two() { .. }

    // async function declaration 
    async function three() { .. }

    // async generator function declaration 
    async function *four() { .. }

    // named function export declaration (ES6 modules)
    export function five() { .. }

And here are some more of the many function expression forms:

    // IIFE
    (function { .. })();
    (function namedIIFE(){ .. });

    // asynchronous IIIFE 
    (async function(){ .. })();
    (async function namedAIIFE(){ .. })();

    // arrow function expressions 
    var f; 
    f = () => 42;
    f = x => x * 2;
    f = (x) => x * 2;
    f = (x,y) => x * y;
    f=x => ({ x:x*2});
    f=x => {returnx*2; };
    f = async x => {
        var y = await doSomethingAsync(x);
        return y * 2;
    };
    someOperation( x => x * 2);
    // ..

Arrow function expressions are syntactically anonymous, meaning the syntax doesn't provide a way to provide a direct name identifier for the function. The function expression may get an inferred name, but only if it's one of the assignment forms, not in the form of being passed as a function call argument. 

Functions can also be specified in class definitions and object literal definitions. They're typically referred to as "methods" when in these forms. 

## Coerive Conditional Comparison 

Here, we refer to conditionl expressions needing to perform coercion-oriented comparisons to make their decisions. 

`if` and `? :` -ternary statements, as well as the test clauses in `while` and `for` loops, all perform an implicit value comparison. Is it strict or coercive? The answer is both: 

    var x = 1;

    if (x) {
        // will run!
    }

    while (x) {
        // will run, once!
    }

You might think of these (x) conditional expressions like this: 

    var x = 1; 

    if (x == true) {
        // will run!
    }    

    while (x == true){
        // will run, once!
        x = false;
    }

In this specific case, that mental model works, but it's not accurate more broadly. Consider:

    var x = "hello";

    if (x) {
        // will run!
    }

    if (x == true){
        // won't run 
    }

So what is the `if` statement actually doing? This is the more accurate mental model: 

    var x = "hello";

    if (Boolean(x) == true) {
        // will run
    }

    // which is the same as: 

    if (Boolean(x) === true) {
        // will run
    }

The `Boolean(..)` function always returns a value of type boolean. The important part is to see that before the comparison, a coercion occurs, from whatever type x currently is, to boolean. 