---
layout: site
title: What do we want from a web programming language, anyway?
subtitle:
---
There's been a [lot](https://gist.github.com/1277224) [of](http://webreflection.blogspot.com/2011/10/what-is-wrong-about-17259-lines-of-code.html) [angst](http://www.nicollet.net/2011/10/lets-whine-about-google-dart/) about [Google Dart](http://www.dartlang.org/) amongst programmers since it was revealed to the public a few days ago.
 
Dart positions itself as a more optimizable Javascript, but has an atrocious Dart-to-Javascript compiler. It is true that this compiler will probably undergo its own optimizations, and that Dart's added capabilities implies at least some overhead when run in Javascript, but this is true of essentially any program automatically translated from one language into another.
 
There are more serious complaints about the language, however, such as a type system that doesn't really work providing the worst of both dynamic and static languages, in my personal opinion, since the [JIT](http://en.wikipedia.org/wiki/Just-in-time_compilation) cannot be sure a particular variable will ever be assigned the wrong type of data, but the programmer still has to annotate the data they're working with.
 
There are some minor complaints with Dart, as well, such as hardwiring the Java object factory pattern into a language that doesn't really explicitly need it, but this is roughly equivalent to Javascript's redundant *new* keyword since you can modify the inheritance structure of an object directly and have the constructor function return that object.
 
The concurrency model, [Isolate](http://www.dartlang.org/docs/api/Isolate.html#Isolate::Isolate), seems heavily-influenced by [Web Workers](http://www.whatwg.org/specs/web-apps/current-work/multipage/workers.html), as well, by making it so the parallel code is highly isolated and can only message pass back-and-forth. I find the [W16 experiment](https://github.com/sheremetyev/w16) using [Software Transactional Memory](http://en.wikipedia.org/wiki/Software_transactional_memory) with parallel [event loops](http://en.wikipedia.org/wiki/Event_loop) much more interesting, because for most cases it will make fairly linear event-driven code scalable (only pathological cases where the code is dependent upon the event dispatch order from unconnected events would this fail to produce a speed-up, but I believe most Javascript code is not written in that way).
 
Clearly, Dart-as-a-language, not just Dart-as-a-language-cross-compiled-into-Javascript, has warts, but are they more or less than Javascript?

Well, doesn't that depend on what we want to use it for? The major complaints against Javascript are:

* Poor optimization target because any variable can contain anything.
* Inability to automatically eliminate dead code because functions can be called dynamically.
* Hard to organize because of no built-in way to require libraries. (The C language doesn't have any defined way to do so either, but the C pre-processor was developed to handle this and is essentially standard; web browsers never exposed a good way to do this.)
* Language will do nothing to stop you doing something stupid, and won't crash when something stupid is done (such as trying to divide a boolean by a string).

But look at these complaints from another light:

* Rapid prototyping because a variable can contain anything.
* Dynamic function calls greatly reduce the code needed to allow customizable behavior of the code ("native-looking" RPC calls by assigning an RPC anonymous function to each desired function name defined from an array)
* No pre-processor magic causing code to do non-obvious things.
* A bug in your javascript is less costly because it won't cause the website to crash.

The reality is that there's a whole spectrum of things that we want from a programming language, which is why we have more than one language out there! Restrictions of certain sorts can make reasoning about a program in certain ways simpler, and can reduce code size for particular kinds of tasks.
 
As others have argued, perhaps that means we should just standardize on a bytecode to run on a virtual machine inside of the browser and let any language that can compile into that bytecode be used by the developers. There are advantages here, as certain languages can put in restrictions they deem important for understanding, such as the near-total elimination of [function side-effects](http://en.wikipedia.org/wiki/Side_effect_%28computer_science%29) from Haskell, or the [elimination of blocking calls](http://en.wikipedia.org/wiki/Asynchronous_I/O) from the API in Node.js. But I feel that breaks the spirit of the web where the code run on the client is fully accessible and auditable, letting users learn from others and become full participants, as well.
 
Further, allowing multiple languages in the browser will lead to more severe fragmentation in capabilities and probably a worsening of the functionality of the web when the lowest-common-denominator is reduced to whatever subset of the language you chose is supported across browsers.
 
So, if we're to replace Javascript with Dart, what changes should we make to Dart's specification to better handle both extremes of strongly and weakly enforced structure, ideally allowing a transition from weak-to-strong as the prototyping is wrapping up and the functionality is becoming more rigidly defined? My opinion:
 
1. **Get rid of optional typing, and replace it with *both* strong and weak typing.* ***If the typing was strictly enforced, the JIT could quickly generate optimal assembly to perform the desired actions upon that variable, but then any non-standard type, such as boolean + null, would have to have a full-blown Dart class and object construction, which is probably slower than what V8 does with a variable that, say, starts out null and is then defined as a boolean. If Dart allowed a special "var" type that accepts anything just like Javascript, you could start out with everything being "var" and then go back and redefine it to "int" or "float" based on how you ended up actually using it. A Dart IDE could then show the developer which type the variable is when they mouse-over the variable name, and a Dart linter could warn about var variables present. If you explicitly want something where the type of variable can change (IE, you want to do duck typing on a set of objects rather than use generics) a convention in the linter to ignore any variable starting with "#" could be used, similarly to how "_" is used to denote private components of a library, so portions of the code that truly ought to be dynamic are easy to identify.
2. **Unify the function types into one.* ***One thing Javascript gets right is that functions are first-class and can be passed around at will. The distinction between anonymous functions with an implicit return and named functions with explicit return will only confuse things.
3. **Allow dynamic function calling, but not by default.* ***This capability, *Object["funcName"]()*, allows for very concise code in Javascript, but makes dead-code optimization impossible and is more prone to screw-ups by a mediocre developer, so it shouldn't be on by default, but should be allowed for a prototyping stage. I propose a syntax such as *allow("dynamicFn"); *that can only be called from the *main()* function of the Dart source. The parameters passed into it can only allow more permissive code, not restrict it, such that libraries will be pushed towards the highest-performing, widest-possible-usage scenario (as they would cause errors if included by code that does not allow these things). I think this is a better idea than Javascripts' *"use strict";*
4. **Allow compilation of Dart to shorter, unsafe Javascript, if desired.* ***Because Dart will exist as a compilation into Javascript for some time, like CoffeeScript, an option to forgo the type safety of Dart for faster real-world performance should be allowed, since the Dart compiler should be catching these type issues ahead of time, anyways, providing most of the advantage over Javascript immediately, while the developer would attempt to do this sort of "straight port" themselves if the performance/bandwidth difference truly matters for their application.
 
A less obfuscated compilation into Javascript would also allow the mixing of Dart and Javascript code in the same codebase, rather than Javascript being only a target. This would help increase Dart adoption since the jump into the language would not be as steep.

That's my opinion on the matter. I've tried to be fair and as unbiased as I can towards Dart, with my suggestions focused primarily at getting a better middle ground between the easy prototyping of Javascript and the more solid assurances of Java, without affecting the verbosity of Dart (which I think is already a nice middle ground).