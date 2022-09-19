# Chapter 2: Illustrating Lexical Scope 

This chapter will illustrate scope with several metaphors. The goal here is to think about how your program is handled by the JS engine in ways that more closely align with how the JS engine actually works. 

##  Marbles, and Buckets, and Bubbles... Oh My!

Imagine you come across a pile of marbles, and notice that all the marbles are colored red, blue, or green. Let's sort all the marbles, dropping the red ones into a red bucket, green into a green bucket, and blue into a blue bucket. After sorting, when you later need a green marble, you already know the green bucket is where to go get it. 

In this metaphor, the marbles are the variables in our program. The bucket are scopes (functions and blocks), which we just conceptually assign individual colors for our discussion purposes. The color of each marble is thus determined by which color scope we find the marble originally created in. 

Consider: 

    //outer/global scope: RED

    var students = [
        { id: 14, name: "Kyle" },
        { id: 73, name: "Suzy" },
        { id: 112, name: "Frank" },
        { id: 6, name: "Sarah" }
    ];

    function getStudentName(studentID) {
        // function scope: BLUE

        for (let student of students) {
            // loop scope: GREEN

            if (student.id == studentID) {
                return student.name; 
            }
        }
    }

    var nextStudent = getStudentName(73);
    console.log(nextStudent);               // Suzy

We've designated three scope colors with code comments:  

1. RED - Outermost global scope
2. BLUE - Scope of funcction `getStudentName(..)`
3. GREEN - Scope of/inside the `for loop`. 

Scope bubbles are dtermined during compilation based on where the functions/blocks of scope are written, the nesting inside each other, and so on. Each scope bubble is entirely contained within its parent scope bubble. 

As the JS engine processes a program (during compoilation), and finds a declaration for a variable, it's asking "Which color scope am I currently in?" The variable is designated as that same color, meaning it belongs to that bucket/bubble.

References to variables/identifiers are allowed if there's a matching declaration either in the current scope, or any scope above/outside the current scope, but not with declarations from lower/nested scopes. 

An explresion in the RED bucket only has access to RED marbles, not BLUE or GREEN. An expression in the BLUE bucket can reference either BLUE or RED marbles, not GREEN. And an expression in the GREEN bucket has access to RED, BLUE, and GREEN marbles. 

Take-aways from marbles & buckets: 

- Variables are declared in specific scopes, which can be thought of as colored marbles from matching-color buckets.
- Any variable reference that appears in the scope where it was declared, or appears in any deeper nested scopes, will be labeled a marble of that same color, unless an intervening scope "shadows" the variable declaration. 
- The determination of colored buckets, and the marbles they contain, happens during compilation. This information is used for variable "lookups" during code execution. 

##  A Conversation Among Friends

Let's meet the members of the JS engine that will have conversations as they process our program: 

- Engine: responsible for start-to-finish compilation and execution of our JavaScript program. 
- Compiler: one of Engine's friends; handles all the dirty work of parsing and code-generation
- Scope Manager: another friend of Engine, collects and maintains a lookup list of all the declared variables/identifiers, and enforces a set of rules as to how these are accessible to currently executing code. 

We have to think like Engine and friends think, ask the questions they ask, and answer their questions likewise. 

Consider: 

    var students = [
        { id: 14, name: "Kyle" },
        { id: 73, name: "Suzy" },
        { id: 112, name: "Frank" },
        { id: 6, name: "Sarah" }
    ]; 

    function getStudentName(studentsID) {
        for (let student of students) {
            if (student.id == studentID) {
                return student.name;
            }
        }
    }

    var nextStudent = getStudentName(73);

    console.log(nextStudent);
    // Suzy

Our focus here will be on the `var students = [ .. ]` declaration and initialization assignment parts. 

Js treats these as two distinct operations, one which Compiler will handle during compilation, and the other which Engine will handle during execution. 

The first thing Compiler will do with this program is perform lexing to break it down into tokens, which it will then parse into a tree (AST).

Once Compiler gets to code generation, there's more detail to consider than may be obvious.  

Here's the steps Compiler will follow to handle that statement: 

1. Encountering `var students`, Compiler will ask Scope Manager to see if a variable named students already exists for that particular scope bucket. If so, Compiler would ignore this declaration and move on. Otherwise, Compiler will produce code that (at execution time) asks Scope Manager to create a new variable called `students` in that scope bucket. 
2. Compiler then produces code for Engine to later execute, to handle the `students = []` assignment. The code Engine runs will first ask Scope Manager if there is a variable called `students` accessible in the current scope bucket. If not, Engine keeps looking elsewhere. Once Engine finds a variable, it assigns the reference of the `[ .. ]` array to it. 

In conversational form: 

<div align="center"><strong>Compiler</strong>: Hey, Scope Manager (of the global scope), I found a formal declaration for an identifier called <code>students</code>, ever heard of it?</div>

<div align="center"><strong>Global (Scope Manager)</strong>: Nope, never heard of it, so I just created it for you.</div>

<div align="center"><strong>Compiler</strong>: Hey, Scope Manager, I found a formal declaration for an identifier called <code>getStudentsName</code>, ever heard of it?</div>

<div align="center"><strong>Global (Scope Manager)</strong>: Nope, but I just created it for you.</div>

<div align="center"><strong>Compiler</strong>: Hey, Scope Manager, <code>getStudentName</code> points to a function, so we need a new scope bucket.</div>

<div align="center"><strong>(Fuction) Scope Manager</strong>: Got it, here's a scope bucket.</div>

<div align="center"><strong>Compiler</strong>: Hey, Scope Manager (of the function), I found a formal parameter declaration for <code>studentID</code>, ever heard of it?</div>

<div align="center"><strong>(Function) Scope Manager</strong>Nope, but now it's created in this scope.</div>

<div align="center"><strong>Compiler</strong>: Hey, Scope Manager (of the function), I found a <code>for-loop</code> that will need its own scope bucket</div>

<div align="center">...</div>

The conversation is a question-and-answer exchange, where **Compiler** asks the current Scope Manger if an encountered identifier declaration has already been encountered. If "no", Scope Manager creates that variable in that scope. If the answer is "yes", then it's effectively skipped over since there's nothing more for that Scope Manager to do. 

The Compiler also signals when it runs across functions or block scopes, so that a new scope bucket and Scope Manager can be instantiated. 

When it comes to execution, the conversation will shift to Engine and Scope Manager: 

<div align="center"><strong>Engine</strong>: Hey, Scope Manager (of the global scope), before we begin, can you look up the identifier <code>getStudentsName</code> so I can assign this function to it?</div>

<div align="center"><strong>(Global) Scope Manager</strong>: Yes, here's the variable.</div>

<div align="center"><strong>Engine</strong>: Hey, Scope Manager, I found a target reference for <code>students</code>, ever heard  of it?</div>

<div align="center"><strong>(Global) Scope Manager</strong>: yes, it was formally declared for this scope, so here it is.</div>

<div align="center"><strong>Engine</strong>: Thanks, I'm initializing <code>students</code> to <code>undefined</code> so it's ready to use.</div>

<div align="center">Hey, Scope Manager (of the global scope), I found a target reference for <code>nextStudent</code>, ever head of it?</div>

<div align="center"><strong>(Global) Scope Manager</strong>: Yes, it was formally declared for this scope, so here it is.</div>

<div align="center"><strong>Engine</strong>: Thanks, I'm initializing <code>nextStudent</code> to <code>undefined</code>, so it's ready to use.</div>

<div align="center">Hey Scope Manager (of the global scope), I found a source reference for <code>getStudentName</code>, ever heard of it?</div>

<div align="center"><strong>(Global Scope Manager)</strong>: Yes, it was formally declared for this scope. Here it is.</div>

<div align="center"><strong>Engine</strong>: Great, the value in <code>getStudentName</code> is a function, so I'm going to execute it.</div>

<div align="center"><strong>Engine</strong>: Hey, Scope Manager, now we need to instantiate the function's scope.</div>

<div align="center">...</div>

This conversation is another question-and-answer exchange, where Engine first asks the current Scope Manager to look up the hoisted `getStudentName` identifier, so as to associate the functino with it. Engine then proceeds to ask Scope manager about the target reference for `students`, and so on. 

To review and summarize how a statement like `var students = [ .. ] is processed`, in two distinct steps: 

1. Compiler sets up the declaration of the scope variable. 
2. While Engine is executing, to process the assignment part of the statement, Engine asks Scope Manager to look up the variable, initializes it to `undefined` so it's ready to use, and then assigns the array value to it. 

## Nested Scope 

The function scope for `getStudentName()` is nested inside the global scope. The block scope of the `for`-loop is similarly nested inside that function scope. Scopes can be lexically nested to any arbitrary depth as the program defines. 

Each scope gets its own Scope Manager instance each time that scope is executed. Each scope automatically has all its identifiers registered at the strt of the scope being executed (variable hoisting).

At the beginning of a scope, if any identifier came from a `function` declaration, that variable is automatically initialized to its associated function reference. And if any identifier came from a `var` declaration (as opposed to `let`/`const`), that variable is automatically initialized to `undefined` so that it can be used; otherwise, the variable remains uninitialized and cannot be used until its full declaration-and-initialization are executed. 

One of the key aspects of lexical scope is that any time an identifier reference cannot be found in the current scope, the next outer scope in the nesting is consulted; that process is repeated until an answer is found or there are no more scopes to consult. 

## Lookup Failures 

When Engine exhausts all lexically available scopes (moving outward) and still cannot resolve the lookuop of an identifier, an error condition then exists. However, depending of the program mode, and the role of the variable (target vs source), this error condition will be handled differently. 

## Undefined Mess

If the variable is a source, an unresolved identifier lookup is considered an undeclared variable, which always results in a `ReferenceError` being thrown. If the variable is a target, and the code at that moment is running in strict-mode, the variable is considered undeclared and similaryly throws a `ReferenceError`.

The error message for an undeclared variable condition, in most JS environments, will look like, "Reference Error: XYZ" is not defined. The phrase "not defined" seems almost identical to the word "undefined" as far as the English language goes. But these two are very differente in JS, and this error message unfortunately creates a persistent confusion. 

"Not defined" really means "not declared", as in a variable that has no matching formal declaration in any lexically available scope. "Undefined" really means a variable was found, but the variable otherwise has no other value in it at the moment, so it defaults to the `undefined` value. 

To make things worse, JS's `typeof` operator returns the string "`undefined`" for variable references in either state: 

    var studentName; 
    typeof studentName;         // "undefined"

    typeof doesntExist;         // "undefined"

Unfortunately, JS developers just have to pay close attention to not mix up which kind of "undefined" they're dealing with!

## Global... What!?

If the variable is a target and strict-mode is not in effect, a confusing and surprising legacy behavior kicks in. The troublesome outcome is that the global scope's Scope Manager will just create an accidental global variable to fulfull that target assgnment. 

Consider: 

    function getStudentName(){
        // assignment to an undeclared variable 

        nextStudent = "Suzy";
    }

    getStudentName();

    console.log(nextStudent);
    // "Suzy" -- oops, an accidental-global variable. 

This sort of accident is a great example of the beneficial protections offered by strict-mode, and why it's such a bad idea not to be using strict-mode, and why it's such a bad idea not to be using strict-mode. In strict-mode, the Global Scope Manager would instead have responded: 

<div align="Center"><Strong>(Global) Scope Manager</strong>: Nope, never heard of it. Sorry, I've got to throw a <code>ReferenceError</code> here.</div>

Never rely on accidental global variables. Always use strict-mode, and always formally declare your variables.

## Building On Metaphors
 
Another metaphor for nested scope resolution, is an office building. 

The building represents our program's nested scope collection. The first floor of the building represents the currently executing scope. the top level of the building is the global scope. 

You resolve a target or source variable reference by first looking on the current floor, and if you don't find it, taking the elevator to the next floor, looking there, then the next, and so on. Once you get to the top floor (the global scope), you either find what you're looking for, or you don't. But you have to stop regardless. 