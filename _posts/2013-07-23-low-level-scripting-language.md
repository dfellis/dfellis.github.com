---
layout: site
title: Low-Level Scripting Language
subtitle: A Fusion of C and Javascript
---
# Low-Level Scripting Language

This post is a midnight braindump of an idea for a low-level scripting language that marries many of the good parts of C and Javascript (and sprinklings of other languages) together, at least in my opinion. It's not fully fleshed out and may turn out to be a fever dream of impossibility come the morning, but I think it may very well be implementable in LLVM (hence the LLSL quasi-title) with most of the speed of C and most of the ease of Javascript.

Here's a list of desired features:

* Four core data types: bytes, structs, arrays, functions. Yes first class functions. All data are bytes, with other data types being syntactic sugar, mostly. Strings for instance would be a struct of function methods and a byte array for the raw data. Arrays are all pascal arrays with a 64-bit length. 2^64 bytes should be enough for everybody. ;)
* ``var`` keyword for creating any of those data types, but static and *must* be initialized with a value.
* Functions are closures by default. Variables are all pointers where the first 64-bits is a reference counter and the space after those 8 bytes is where the data itself resides. This may be silly memory usage for single byte values, but means that math can all be done with simple pointer offsets (fixed offsets, at that, so simpler to inline) while variables are automatically reference counted and only variables within the currently-ending scope have to be checked for removal.
* No main function, just start writing statements. Since closures are possible the outermost scope is essentially a virtual main function.
* Modules because fuck includes.
* Operators are just functions. Any valid function name is also an operator name, so ``1 + 2`` is ``+(1, 2)``. Ternary operators not allowed. Operator syntax is just syntactic sugar.
* Functions can be overloaded by type. How this will work with functions being assigned to variables is not a solved problem, but being unable to overload means we can't define an adder for non-numeric values, such as a BigInt struct/object.
* Functions use a very terse syntax: ``var func = (arg1, arg2, ...) => statementOrFunction, statementOrFunction;``
* ASM/LLVM opcodes directly usable in the language with a backtick syntax. This is actually the only "statement", besides value literals, everything else in the standard library will be built from it.
* ``if`` is just a function that takes three functions, the conditional function, the true function, and the false function. Implemented with whatever LLVM's ``jumpIfZero`` and ``jumpIfNotZero`` opcodes are.
* All functions that do not return a function can be inlined. Therefore no extra scope needs to be made in those cases. Should it be that aggressive on inlining?
* I'm thinking yes, because closures and automatic reference counting means each new function call probably needs to create a copy of the scope, add in its own scoped variables, and increment all of the reference counts, then on exit decrement all of its reference counts, free anything that hit zero, and destroy its scope.
* Functions return their last statement. There is no return keyword.
* A HashTable *will* be in the standard library
* A module registry is vitally important for any modern language to grow. The standard library is simply another module that can be depended on. Only the ``require`` method has to absolutely be present for loading code, and is actually a compiler construct.
* Quotes are Javascript style, ``"`` and ``'`` are identical, just changing which character needs escaping.
* ``&`` is used to get the value of pointer of a variable. Useful in the inline-ASM syntax. May be restricted to within it like ``\`` general is with escaping strings.
* ``#`` is the comment symbol, because fuck the ``//`` ``/* */`` holy wars, and easier to use as a script with the interpreter (if my guess is right, compilation for this thing should be faster than JVM startup times, most of the time)
* Keywords and symbols: `` ` ``, ``var``, ``byte``, ``struct``, ``=``, ``=>``, ``{``, ``}``, ``[``, ``]``, ``(``, ``)``, ``require``, ``export``, ``"``, ``'``, ``.``, ``,``, ``;``, ``&``, ``#``. That's it!

Possible example code:


    var print = require('print');
    
    var helloWorld = 'Hello, World!';
    
    print(helloWorld);

Simple hello world exercise looks identical to node.js

    var print = require('print');
    
    var < = (a byte, b byte) => `LT &a &b`;
    
    print(8 < 3); # Prints 0

Defining a comparison operator with inline LLVM about as easy as the Hello, World.

    var print = require('print');
    
    var myStruct = {
        foo = 'bar',
        baz = [ 123, 456 ]
    };
    
    print(myStruct.foo); # Prints bar

Structs look JSON-ish. Debating reuse of equal sign, but would like to keep ``:`` free for custom operators.

    var print = require('print');
    var HashTable = require('HashTable'); # Functions that create structs should be capitalized
    
    # Not sure if the following will be possible
    var myHash = HashTable({
        foo = 'bar',
        baz = 123
    }); # HashTable can be initialized with a struct and auto-boxes the values (somehow)
    
    myHash.set('newValue', 'someValue'); # This will only work with the desired operator overloading
    
    print(myHash.get('newValue')); # Prints someValue

There are some kinks to figure out with actual hash tables so they aren't totally Java-y, but this seems feasible to my sleep-deprived brain. Perhaps need a ``typeof`` keyword, too.

## Update on stories for my son

This past month has not been good for story writing. Lots of extra work, plus hosting a guest for the 4th one week, my son's birthday another, his birthday party this past weekend... I intend to have the story written by this weekend and be back on track, though.