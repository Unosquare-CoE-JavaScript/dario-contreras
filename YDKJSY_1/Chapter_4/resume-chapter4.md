# Chapter 4: The Bigger Picture

## Pillar 1: Scope and Closure

The organization of variables into units of scope is one of the most foundational characteristics of any language. Scopes are like buckets, and variables are like marbles you put into those buckets. 

Scopes nest inside each other, and for any given expression or statement, only variables at that level of scope nesting, or in higher/outer scopes, are accessible; variables from lower/inner scopes are hidden and anaccesible. 

This is how scopes behave in most languages, which is called lexical scope. The scope unit boundaries, and how variables are organized in them, is determined at the time the program is parsed. 

JS is lexically scoped, though many claim it isn't, because of two particular characteristics of its model that are not present in other lexically scoped languages. 

The first is called hoisting. Hoisting happens when all variables declared anywhere in a scope are treated as if they're declared at the beginning of the scope. The other is that `var`-declared variables are function scoped, even if they appear inside a block.

Closure is a natural result of lexical scope whn the langage has functions as first-class values, as JS does. Whe a function makes reference to variables from an outer scope, and that function is passed around as a value and executed in other scopes, it maintains access to its original scope variables; this is closure. 

## Pillar 2: Prototypes

JS is one of very few languages where you have the option to create objects directly and explicitly, without first defining their structure in a class. 

Classes are just one pattern you can build on top of such power. But another approach is to simply embrace objects as objects, forget classes altogether, and let objects cooperate through the prototype chain. This is called behavior delegation. 

## Pillar 3: Types and Coercion

No JS program will do anything useful if it doesn't properly leverage JS's value types, as well as the conversion (coercion) of values between types. 

