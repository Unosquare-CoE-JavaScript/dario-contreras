# Chapter 1: What's the scope? 

## Compiled vs Interpreted

Code compilation is a set of steps that process the text of your code and turn it into a list of instructions the computer can understand.

Interpretation performs a similar task to compilation, in that it transforms your program into machine-understandable instructions. But the processing model is different. With interpretation, the source code is transformed line by line; each line or statement is executed before immediately proceeding to processing the next line of the source code. 

Modern JS engines employ numerous variations of both compilation and interpretation in the handlings of JS programs. The assertion in this book, is that JS is most accurately portrayed as a compiled language. 

## Compiling Code

Why does it even matter whether JS is copmiled or not? 

Scope is primarily determined during compilation, so understanding how compilation and execution relate is key in mastering scope. 

A program is processed by a compiler in three basic stages: 

1. **Tokenizing/Lexing**: breaking up a string of characters into meaningful (to the language) chunks, called tokens.  Consider the program `var a = 2;`. This program would likely be broken up into the following tokens: `var, a, =, 2,` and `;`. Whitespace may or may not be persisted as a token, depending on whether it's meaningful or not. Note: The difference between tokenizing and lexing is subtle and academic, but it centers on whether or not these tokens are identified in a stateless or stateful way. In other words, if the tokenizer were to invoke stateful parsing rules to figure out whether `a` should be considered a distinct token or just part of another token, that would be lexing. 

2. **Parsing**: taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This is called an Abstract Syntax Tree (AST). For example, the tree for `var a = 2`; might start with a top-level node called `VariableDeclaration`, with a child node called `Identifier` (whose value is `a`), and another child called `AssignmentExpression` which itself has a child called `NumericLiteral` (whose value is 2).  

3. **Code Generation**: taking an AST and turning it into executable code. The JS engine takes the just described AST for `var a = 2`; and turns it into a set of machine instructions to actually create a variable called `a` (including reserving memory, etc.), and then store a value into `a`.

The JS engine is vastly more complex than just the stages. In fact, code can even be recompiled and re-optimized during the progression of execution. 

## Required: Two Phases

The most important observation we can make about processing of JS programs, is that it occurs in at least two phases: parsing/compilation first, then execution. While the JS specification does not require "compilation" explicitly, it requires behavior that is essentially only practical with a complile-then-execute approach. 

Things that you can observe to prove this to yourself: 

1. Syntax errors
2. Early errors
3. Hoisting

## Syntax Errors from the Start

Consider: 

    var greeting = "Hello"; 

    console.log(greeting);

    greeting = ."Hi";
    // SyntaxError: unexpected token. 

This program throws a `SyntaxError` about the unexpected . token right before the "Hi" string. Since the syntax error happens after the well-formed `console.log(..)` statement, if JS was executing top-down line by line, one would expect the "Hello" message being printed before the syntax error being thrown. That doesn't happen. 

In fact, the only way the JS engine could know about the syntax error on the third line, before executing the first and second lines, is by the JS engine first parsing the entire program before any of it is executed. 

## Early Errors

Consider:

    console.log("Howdy");

    saySomething("Hello", "Hi");
    // Uncaught SyntaxError: Duplicate parameter name not
    // allowed in this context 

    function saySomething(greeting, greeting) {
        "use strict";
        console.log(greeting);
    }

The "Howdy" message is not printed, despite being a well-formed statement. 

Just like the snippet in the previous setion, the `SyntaxError` here is thrown before the program is executed. In this case, it's because strict-mode forbids, among many other things, functions to have duplicte parameter names; this has always been allowed in non-strict-mode. 

The only reasonable explanation on why JS knows this before that piece of code is executed, is because the code must first be fully parsed before any execution occurs. 

## Hoisting 

Finally, consider: 

    function saySomething() {
        var greeting = "Hello";
            {
                greeting = "Howdy";         // error comes from here
                let greeting = "Hi";
                console.log(greeting);
            }
    }

    saySomething();
    // ReferenceError: Cannot access 'greeting' before initialization

What's happening is that the `greeting` variable for that statement belongs to the declaration on the next line, `let greeting = "Hi"`, rather than to the previous `var greeting = "Hello"`.

The only way the JS engine could know, at the line where the error is thrown, that the next statement would declare a block-scoped variable of the same name (`greeting`) is if the JS engine had already proessed this code in an earlier pass, and already set up all the scopes and their variable associations. This processing of scopes and declarations can only accurately be accomplished by parsing the program before execution. 

The `ReferenceError` here technically comes from `greeting = "Howdy"` accessing the `greeting` variable too early, a conflict referred to as Temporal Dead Zone (TDZ).

To wrap this up, we can conclude that what the engine is doing while processing JS programs, is much more alike compilation than not. 

Classifying JS as a compiled language is about keeping a clear distinction in our minds about the phase where JS code is processed and analyzed; this phase observably and indisputedly happens before the code starts to be executed. 

## Compiler Speak 

Let's turn our attention to how the JS engine identifies variables and determines the scopes of a program as it is compiled.  

First, let's examine a simple JS program to use for analysis over the next several chapters: 

    var students = [
        { id: 14, name: "Kyle" },
        { id: 73, name: "Suzy" },
        { id: 112, name: "Frank" },
        { id: 6, name: "Sarah" }
    ];

    functiongetStudentName(studentID) {
        for (let student of students) {
            if (student.id == studentID) {
                return student.name;
            }
        }
    }

    var nextStudent = getStudentName(73);

    console.log(nextStudent);
    // Suzy

Other than declarations, all occurrences of variables/identifiers in a program serve in one of two "roles": either they're the target of an assignment or they're the source of a value. 

How do you know if a variable is a target? Check if there is a value that is being assigned to it; if so, it's a target. If not, then the variable is a source. 

For the JS engine to properly handle a program's variables, it must first label each occurrence of a variable as target or source. We'll dig in now to how each role is determined. 

## Targets 

What makes a variable a target? Consider: 

    students = [ //.. 

This statement is clearly an assignment operation, remember, the `var students` part is handled entirely as a declaration at compile time, and is thus irrelevant during execution; we left it out for clarity and focus. Same with the `nextStudent = getStudentName(73)` statement.

There are three other target assignment operations in the code that are perhaps less obvious. One of them: 

    for (let student of students) {}

That statement assigns a value to `student` for each iteration of the loop. Another target reference: 

    getStudentName(73)

But how is that an assignment to a targe? Look closely: the argument `73` is assigned to the parameter `studentID`.

The last one is: 

    function getStudentName(studentID) {

A function declaration is a special case of a target reference. 

## Sources 

In `for (let student of students)`, we said that `student` is a target but `students` is a source reference. 

In the statement `if (student.id == studentID)`, both `student` and `studentID` are source references. `student` is also a source reference in `return student.name`.

In `getStudentName(73)`, `getStudentName` is a source reference which we hope resolves to a function reference value. In `console.log(nextStudent)`, `console` is a source reference, as is `nextStudent`.

## Cheating: Runtime Scope Modifications 

In non-stric mode, there are two ways to a program's scope during runtime. Neither of these techniques should be used, but it's important to be aware of them in case you run across them in some programs. 

The `eval(..)` function receives a string of code to compile and execute on the fly during the program runtime. If that string of code has a `var` or `function` declaration in it, those declarations will modify the current scope that the `eval(..)` is currently executing in: 

    function badIdea(){
        eval("var oops = 'Ugh!");
        console.log(oops);
    }

    badIdea(); // Ugh!

If the `eval(..)` had not been present, the `oops` variable in `console.log(oops)` would not exist, and would throw a `ReferenceError`. But `eval(..)` modifies the scope of the `badIdea()` function at runtime. This is bad for many reasons, including the performance hit of modifying the already compiled and optimized scope, every time `badIdea()` runs. 

The second cheat is the `with` keyword, which essentially dynamically turns an object into a local scope.

Consider: 

    var badIdea = { oops: "Ugh!" };

    with (badIdea) {
        console.log(oops);  // !Ugh
    }

The global scope was not modified here, but `badIdea` was turned into a scope at runtime rather than compile time, and its property `oops` becomes a variable in that scope. Terrible idea for performance and readability reasons. 

## Lexical Scope

JS's scope is determined at compile time; the term for this kind of scope is "lexical scope". "Lexical" is associated with the "lexing" stage of compilation.

The key idea of "lexical scope" is that it's controlled entirely by the placement of function, blocks, and variable declarations, in relation to one another.

If you palce a variable declaration inside a function, the compiler handles this declaration as it's parsing the function, and associates that declaration with the function's scope. If a variable is block-scope declared (`let` / `const`), then it's associated with the nearest enclosing `{ .. }` block, rather than its enclosing function (as with `var`). 

Furthermore, a reference for a variable must be resolved as coming from one of the scopes that are lexically available to it; otherwise the variable is said to be undeclared, resulting in an error. If the variable is not declared in the current scope, the next outer/enclosing scope will be consulted. this process of stepping out one level of scope nesting continues until either a matching variable declaration can be found, or the global scope is reached and there's nowhere else to go. 

Note that compilation doesn't actually do anything in terms of reserving memory for scopes and variables. None of the program has been executed yet. 

Compilation creates a map of all the lexical scopes that lays out what the program will need while it executes. 

In other words, while scopes are identified during compilation, they're not actually created until runtime, each time a scope needs to run. 