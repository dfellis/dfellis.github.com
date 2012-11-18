---
layout: site
title: Not Optionally-Typed, Gradually-Typed
subtitle:
---
Why was Dart so uniformly rejected by the development community; from both the dynamic-leaning and static-leaning developers?

What was it about Dart that was so distasteful? It has the "classic" object inheritance that essentially everyone but Javascript and Smalltalk developers cried for, while staying much terser than Java and keeping dynamic types available for those who prefer dynamic languages. If I could sum it in just one line, it would be:

    int myBool = "Hello, World";

Don't you just want to punch me in the face with that line?

So, developers have long had to deal with terse or misleading variable names, but they've always been able to rely on the original declaration to either find out just what the variable actually is, or at least have a "``var``" warning sign that you're on your own with figuring out the code.

Dart simply makes it harder to debug source code without help from a debugger, purely by making types optional in their attempt to please both sides of the aisle.

Are static and dynamic languages truly irreconcilable? I don't think so. Programmers are programmers. They may have different styles of programming, but they're all after the goal of automating some task so it can be done far more rapidly than it could ever be done by hand.

So, what's the motive for choosing a static or dynamic language, when external requirements don't demand one or the other?

For those who prefer dynamic languages, I believe the motive is similar to the following:

> I have been tasked with writing a new program. The general goal of the program has been defined, but how exactly it should be accomplished and how the user should use it has been left to me.
> I don't want to use a static language that will force me to write lots of boilerplate code for a particular design that will probably be only half-right and have to be scrapped and rewritten a few times, drastically affecting code all over the program, when I can write in a dynamic language that will let me compose my functions and objects, shoehorn in new features as needed, and massage it into the actual design through a feedback loop of user testing and data structure analysis over time during development, vastly improving ROI for my client.
> If/when a bottleneck in the code appears that requires static typing, I can rewrite that small portion in a static language when I have a well-defined idea of what it's inputs and outputs are, so I really can optimize it well, and don't have to rewrite it over and over.

While for those who prefer static languages, the motive is probably like the following:

> I have been tasked with writing a new program. The general goal of the program has been defined, but how exactly it should be accomplished and how the user should use it has been left to me.
> I don't want to use a dynamic language that will force me to write lots of boilerplate code to make sure my data input is the right type that will probably be only half-right and have to be rewritten a few times during the course of testing each component, when with a static language the compiler can automatically indicate issues with the composability of my functions and objects, add features on top of old without worry, and keep my code so much faster than dynamic code that I only need to worry about my data structures if and only if there is a demonstrated bottleneck demonstrated through a feedback loop of user testing and data structure analysis over time during development, vastly improving ROI for my client.
> If/when the architecture being developed is deemed the bottleneck of the code that requires rewriting, I have a well-defined idea of what it should now look like, so I can really optimize it well, and I can reuse most of the already-typed code without rewriting it from a dynamic language.

The difference between the two types of developers, I think, is whether their approach is top-down or bottom-up (primarily) towards programs. Top-down developers don't want the minutia of the underlying hardware and data structures polluting their exploration of algorithms and architectures for the program, while bottom-up developers don't want the overall architecture distracting them from optimizing the algorithms and data structures that they will eventually build the architecture from.

But both sorts of feedback are important -- for the top-down developer, perhaps the underlying data structures necessary for an architecture can never be optimized into an efficient platform, even when deferred to a static language, and for the bottom-up developer, the data structures developed may never be composable into an efficient overall architecture.

Being able to explore in both directions helps tremendously, but few want to do it because that has traditionally meant "throwaway" programs written in another language to demonstrate feasibility -- code that can't really be re-used. *"If I was just smart enough, I wouldn't have to write that **other** code, and could just write in my preferred language."*

On top of that, clients can mistake the "throwaway" program demonstrated as the real McCoy, and demand development of the program continue until completion -- and then not only could the code be an awful kludge thrown together just to demonstrate some functionality, but it's also written in a language you're not as comfortable in.

A single language that could allow both top-down and bottom-up exploration and development. It would, by most definitions, be considered a dynamic language, but it wouldn't be a "pure" dynamic language like TCL, but some sort of hybrid that allows you to mark certain variables as a fixed type, and restrict what can be done with certain object definitions, like a static language.

Then dynamic variables and objects can be used for top-down exploration of the architecture, while "well-defined" parts of the code can be "solidified" into static types to improve performance and future maintainability, *gradually* typing the source code until completed.

Dart misses that point, but what out there "gets" it? [HaXe](http://haxe.org) is one example of a language that can bridge static and dynamic languages; it can compile to Javascript, Actionscript, Java and C++. (It's too Java-like for my tastes, though, and the fact that it's a language-to-language translation means that you'll always have to deal with unfamiliar error messages for whatever target you're compiling to.)

However, one language particularly important to the Dart developers seems to get it: Javascript.

The latest version of Javascript has moved towards a "gradual typing" system like I've described.

* There are now typed arrays for storing guaranteed integer and double values that can be iterated over and operated on far faster than normal types.
* There are buffer objects, references to raw binary data, that can be passed through Javascript to various APIs such as WebSockets, WebGL, and so on.
* You can now explicitly define objects and "freeze" their definition so the engine can optimize the object type in memory and further improve performance.
* A custom taken from compilers of old to initialize variables on declaration can be used as hints to the JS engine about the type of the variable to speed JIT optimizations.

And that's just from the most recent version (ECMAScript 5) of Javascript. Future enhancements currently in debate would bring in true "classic" inheritance and actual explicit typing (with errors on mismatch).

As pressure is applied to Javascript to do everything traditionally falling upon the shoulders of static languages, it appears that a hard diamond may be forged from the soft coal.

Dart was poorly received because their attempt to please both camps was done by compromising the workflow of both. The future of Javascript may actually succeed by enhancing the workflows of both, instead.