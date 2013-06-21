---
layout: site
title: A more "Node-y" EventEmitter than EventEmitter
subtitle: With a bit of the DOM Events mixed in, too!
---
## Background

Node.js's [EventEmitter](http://nodejs.org/api/events.html) has always struck me as not really fitting in with the rest of the Node.js ethos. Emitting events is a wholly synchronous process, so the delay from ``this.emit`` to the line of code after it will unpredictably vary depending on the number of listeners and how much work they do.

On top of that, EventEmitter is crippled versus the browser's [DOM Events](http://www.w3.org/TR/DOM-Level-2-Events/events.html), which allow event sources to [make certain events cancelable](http://en.wikipedia.org/wiki/DOM_events#Event_object). (It also has an event propagation mechanism up and down the DOM tree, but that's a DOM-specific behavior coupled closely to the very concept of the DOM tree.)

DOM Event Cancelation, like most (all?) of the official Javascript APIs that were originally designed by committee in the 90's, assumes the listeners are synchronous, so if you are going to cancel an event, that you're going to do it in the same Javascript event loop tick as the code that fired the event, so its return value would indicate whether or not to cancel the event.

This is probably why the EventEmitter doesn't allow cancelable events, as all of the rest of the Node.js APIs are asynchronous, so any non-trivial cancelation would be impossible with that design.

But I needed events for [queue-flow](http://dfellis.github.com/queue-flow) and being able to cancel events (especially the ``push`` event) was desirable, so I wrote a buggy, half-implemented version of ~~Lisp~~ EventEmitter that made one modification to ``emit`` -- it takes a callback that gets a boolean indicating whether or not the event should continue. It also re-used code that "guesses" whether or not a given function is synchronous or asynchronous to allow both kinds of listeners to be used.

So I wrote some tests around it and called it a day.

But this event emitter code was pretty big (larger than all the rest of my constructor function) and I was shamelessly copy-pasting it for my derivative constructor functions [sloppy-queue-flow](https://github.com/dfellis/sloppy-queue-flow) and [parallel-queue-flow](https://github.com/dfellis/parallel-queue-flow). It had to be broken out.

At first I considered just ripping the code out as-is into a separate module and calling it a day, but I was already using the standard EventEmitter in [multitransport-jsonrpc](http://uber.github.com/multitransport-jsonrpc), and I decided to reimplement it sticking as close as possible to Node's EventEmitter. Why?

1. Reduced cognitive overhead by keeping the same API.
2. Easier migration path back and forth between them based on the needs of the project (if you already emit events but suddenly have one that you'd like to cancel, you can just replace:

{% highlight js %}
var EventEmitter = require('events').EventEmitter;
{% endhighlight %}

with

{% highlight js %}
var EventEmitter = require('async-cancelable-events');
{% endhighlight %}

and replace

{% highlight js %}
this.emit('eventIWantToCancel', value);
{% endhighlight %}

with

{% highlight js %}
this.emit('eventIWantToCancel', value, function(toContinue) { /* ... */ });
{% endhighlight %}

So, if you find you need this capability in your events, just ``npm install async-cancelable-events``, or grab the minified version for the browser [from the async-cancelable-events repo](https://github.com/dfellis/async-cancelable-events).