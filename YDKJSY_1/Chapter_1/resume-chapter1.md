# Chapter 1: What is JavaScript? 

## What's with that name? 

The name JavaScript is an artifact of marketing shenaningans. Why? Because this language was originally designed to appeal to an audiencie of motly Java programmers, and because the word "script" was popular at the time to refer to lightweight programs. These lightweight "scripts" would be the first ones to embed inside of pages on this new thing called the web. 

In a nutshell, JavaScript was a marketing ploy to try to position this language as a palatable alternative to writing the heavier and more well-known Java of the day.  

The superficial resemblances between JavaScript's and Java's code come from the syntax expectations of developers already familiarized with C. 

Example: '{' and '}' to begin and end a block of code respectively

Nowadays, the official name of the language is ECMAScript. This is specified by TC39 and formalized by the ECMA standards. In other words, the JavaScript/JS that runs in a browser or in Node.js, is an implementation of the ES2022 standard. 

## Language Specification

TC39 is the technical steering committee that manages JS. Their primary task is to manage the official specification for the language, they meet to vote on any agreed changes, which they then submit to ECMA, the standards organization. 

JS's syntax and behavior are defined in the ES specification. 

The TC39 committee is comprised of between 50 and about 100 different people from a broad seaction of web-invested companies, such as browser and device makers.  

All TC39 proposals progress through a five-stage process. From Stage 0 to Stage 4.  

Once a proposal reaches "Stage 4" status, it is eligible to be included in the next yearly revision of the language. It can take anywhere from several months to a few years for a proposal to work its way through these stages.  

This comittee makes sure that you can learn one JS, and rely on that same JS everywhere. 

## The Web Rules Everything About JS 

How JS is implemented for web browsers is, in all practicality, the only reality that matters. 

For the most part, the JS defined in the specification and the JS that runs in browser-based JS engines is the same. But there are some differences that must be considered. 

Sometimes the JS specification will dictate some new or refined behavior, and yet that won't exactly match with how it works in browser-based JS engines. In such cases, the engines will refuse to conform to a specification-dictated change because it would break some web content. 

In these cases, often TC39 will backtrack and simply choose to confrom the specification to the reality of the web, but occasionally, TC39 will decide the specification should stick firm on some point even though it is unlikely that browser-based JS engines will ever conform. 

If you'd like to, you can refer to ECMAScript's Appendix B to checkout the additional features for web browsers, but it's a good idea to avoid these constructs to be future safe.

## Not All (Web) JS...

Various JS environments (like browser JS engines, Node.js, etc.) add APIs into the global scope of your JS programs that five you environment-specific capabilities, like being able to pop an alert-style vox in the user's browser. 

A common example is `console.log(..)` and all the other `console.*` methods. These are not specified in JS but because of their universal utility are defined by pretty much ebery JS environment, according to a roughly agreed consensus.

Most of the cross-browser differeneces people complain about with "JS is so inconsistent!" claims are due to differences in how those environment behaviors work, not in how the JS itself works. 

So a `console.log(..)` call is JS, but `console.log` intself is just a guest, not part of the official JS specification. 

## It's Not Always JS 

Using the console/REPL (Read-Evaluate-Print-Loop) in your browser's Developer Tools (or Node) feels like a straight forward JS environment, it's not. 

The Developer Tools prioritize Developer Experience, but they don't have the goal to accurately and purely reflect all nuances of strict-spec JS behavior. 

The developer console is not trying to pretend to be a JS compiler that handles your entered code exactly the same way the JS engine handles a .js file. It's trying to make it easy for you to quickly enter a few lines of code and see the results immediately.  

Don't trust what behavior you see in a developer console as representign exact to-the-letter JS semantics, instead, think od the console as a "JS-frienly" environment. 

## Many Faces 

The term "paradigm" in programming languages refers to a broad mindset and approach to structuring code. But no matter what a program's individual style may be, the big picture divisions around paradigms are almos always evident at first glance of any program. 

Typical paradigm-level code categories include: 

Procedural style: organizes code in a top-down, linear progression through a pre-determined set of operations,usually collected together in related units called proce-dures.

OO style: organizes code by collecting logic and data together into units called classes.

FP style: organizes code into functions (pure computa-tions as opposed to procedures), and the adaptations ofthose functions as values.

Some languages are heavily slanted toward one pradigm, but many languages also support code patterns that can come from, and even mix and match from different paradimgs.

JavaScript is most definitely a multi-paradigm language. You can write procedural, class-oriented, or FP-style code, and you can make those decisions on a line-by-line basis, instead of being forced into an all-or-nothing choice.

## Backwards & Forwards

JavaScript has backwards compatibility. This means that once something accepted as valid JS, there will not be a future change to the language that causes that code to become invalid JS. In other words, code written in 1995, should still work today.  

JavaScript is not forwards-compatible. Being forwards-compatible means that including a new addition to the language in a program would not cause that program to break if it were run in an older JS engine. 

## Jumping the Gaps 

Since JS is not forwadds-compatible, it means that there is always the potential for a gap between code that you can write that's valid JS, and the oldest engine that your site or application needs to support.  

For new and incompatible syntax, the solution is transpiling. Transpiling is a contrived and community-invented term to describe using a tool to convert the source code of a program from one form to another. 

Typically, forwards-compatibility problems related to syntax are solved by using a transpiler (the most common one being Babel) to convert from that newer JS syntax version to an equivalent older syntax. 

To wrap this up, developers should focus on writing the clean, new syntax forms, and le the tools take care of producing a forwards-compatible version of that code that is suitable to deploy and run on the oldest-supported JS engine environments. 

## Filling the Gaps 

If the forwards-compatibility issue is related to a missing API method that was only recently addes, the most common solution is to provide a definition for that missing API method that stands in and acts as if the older environment had already had it natively defined. This pattern is called a polyfill (aka "shim").

Consider this code: 

    // getSomeRecords() returns us a promise for some
    // data it will fetch

    var pr=getSomeRecords();

    // show the UI spinner while we get the data
    startSpinner();

    pr.then(renderRecords) // render if successful.
    catch(showError) // show an error if not.
    finally(hideSpinner) // always hide the spinner

This code uses an ES2019 feature, the `finally(..)` method on the promidse prototype. If the code were used in a pre ES2019 environment, the `finally(..)` method would not exist, and an error would occur, making a polyfill for `finally(..)` required. 

Transpilers like Babel typically detect which polyfills your code needs and provide them automatically for you. But occasionally you may need to include/define them explicitly.

Transpilation and polyfilling are two highly effective tech-niques for addressing that gap between code that uses the latest stable features in the language and the old environments a site or application needs to still support. 

## What's an interpretation? 

Is JS an interpreted script or a compiled program? Languages regarded as "compiled" usually produce a portable (binary) representation of the program that is distributed for execution later. Since we don't really observe that kind of model with JS, many claim that disqualifies JS from the category.  

All this aside, the real reason it matters to have a clear picture on whether JS is interpreted or compiled relates to the nature of how errors are handled. 

Historically, scripted or interpreted languages were executed in generally a top-down and line-by-line fashion; there's typically not an initial pass through the program to process it before execution begins. In scripted or interpreted languages, an error on line 5 of a program won't be discovered until lines 1 through 4 have already executed. Notably, the error on line 5 might be due to a runtime condition, such as some variable or value having an unsuitable value for an operation, or it may be due to a malformed statement/command on that line. Depending on context, deferring error handling to the line the error occurs on may be a desirable or undesirable effect. 

Compare that to languages which do go through a proccesing step, typically called parsing, before any execution occurs.  On the parsing processing step, an invalid command (such as broken syntax) on line 5 would be caught during the parsing phase, before any execution has begun, and none of the program would run. 
 
All compiled languages are "parsed". In classic compilation theory, the last remaining step after parsing is code generation: producing an executable form. 

Once any source program has been fully parsed, it's very common that its subsequent execution will, in some form or fashion, include a translation from the parsed form of the program to that executable form, so it's not that much of a stretch to say that, in spirit, parsed languages are compiled languages. 

JS source code is parsed before it is executed, but is it compiled?  

The flow of a JS program is: 

1. After a program leaves a developer's editor, it gets transpiled by Babel, then packed by Webpack, then it gets delivered in that very different form to a JS engine. 
2. The JS engine parses the code to an AST. 
3. Then the engine converts that AST to a kind-of byte code, a binary intermediate representation (IR), which is then refined/converted even further by the optimizing JIT complier. 
4. Finally, the JS VM executes the program. 

In spirit, if not in practice, JS is a compiled language. 

## Web Assembly (WASM)

The original intent of WASM was to provide a path for non-Js programs (C, etc.) to be converted to a form that could run in the JS engine. WASM also chose to additionally get around some of the inherent delays in JS parsing/compilation before a program can execute, by representing the program in a form that is entirely unlike JS. 

WASM is a representationf ormat more akin to Assembly (hence, its name) that can be processed by a JS engine by skipping the parsing/compilation that the JS engine normally does. The parsing/compilation of a WASM-targeted program happen ahead of time (AOT); what's distributed is a binary.packed program ready for the JS engine to execute with very minimal processing.

## Strictly Speaking 

In 2009, on the release of ES5, JS added strict mode as an opt-in mechanism for encouraging better JS programs. 

Why strict mode? Strict mode should be think of as a guide to the best way to do things so that the JS engine has the best chance of optimizing and efficiently running the code. Most JS code is worked on by teams of developers, so the strict-ness of strict mode often helps collaboration on code by avoiding some of the more problematic mistakes that slip by in non-strict mode.

Most strict mode controls are in the form of early errors, meaning errors that aren't strctly syntax errors but are still thrown at compile time (before the code is run). For example, strict mode disallows naming two functino parameters the same, and results in an early error. Some other strict mode controls are only observable at runtime, such as how this defaults to undefined instead of the global object. 

Strict mode is switched on per file with a special pragma (nothing allowed before it except comments/whitespace):

    //only whitespace and comments are allowed
    //before the use-strict pragma
    "use strict";
    //the rest of the file runs in strict mode

    Strict mode can alternatively be turned on per-function scope, with exactly the same rules about its surroundings: 

    function someOperations() {
        // whitespace and comments are fine here 
        "use strict";
        // all this code will run in strict mode 
    }

Interestingly, if a file has strict mode turned on, the function-level strict mode pragmas are disallowed. So you have to pick one or the other. 

Is worth nothing that virtually all transpiled code ends up in strict mode even if the original source code isn't written as such. Most JS code in production has been transpiled, so that means most JS is already adhering to strict mode. Moreover, ES6 modules assume strict mode, so all code in such files is automatically defaulted to strict mode. 

Taken together, strict mode is largely the de facto default even though technically it's not actually the default.

## Defined

JS is an implementation of the ECMAScript standard, which is guided by the TC39 committee and hosted by ECMA. It runs in browsers and other JS environments such as Node.js.

JS is a multi-paradigmn and compiled language. 