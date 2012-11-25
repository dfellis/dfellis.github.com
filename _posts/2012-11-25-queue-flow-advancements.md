---
layout: site
title: queue-flow Advancements
subtitle: Subqueues, Parallelism, Promises, and the Future
---
[``queue-flow``](http://dfellis.github.com/queue-flow) is a library I have been working on for a little over 4 months with (as of today) 54 versions published onto [npm](http://npmjs.org), currently at 0.5.30 (yes, 30). For those who want to see what queue-flow is all about, I recommend [viewing the tutorial](http://dfellis.github.com/queue-flow/2012/09/21/tutorial/). This blog post is going to focus on new features developed over the past few days, what they're good for, and where queue-flow is heading in the immediate future.

## subqueues

Code reuse has been a particular pain point for me with queue-flow. While individual steps are easy to reuse as they're simple functions that can be named and then referenced by any particular queue, reusing multiple steps of a queue has required either copy-pasting steps (the easy way) or using tricks with the ``branch`` method and keeping track of meta-data on which queues any particular piece of data should go to at the end of the reused queue, which may not be possible if any particular step is using a third-party library (or require ugly wrapping of that step to continue tracking the metadata).

This was caused by the mechanism by which queues are constructed and used: Each step along the queue is associated with a queue-flow object and most of the queue-flow methods construct a new anonymous queue-flow object that its results are pushed into. This linking between a queue, the processing method, and the next queue in the line is done once: synchronously on initialization.

``branch``, on the other hand, does not construct a queue, but allows static or dynamic linking of two or more already-defined queues, and could be used to simulate a subqueue, but does require more mental gymnastics to do (it looks very much like how functions are actually implemented in assembly languages) and someone not familiar with queue-flow's internals could easily slow down their queues dramatically by reconstructing the subqueue for each value to be processed (queue construction, while I have tried to keep fast, has not been a priority because that cost is easily amortized in common scenarios).

The new ``subqueue`` method takes care of that mess, allows subqueues to be defined asynchronously only once, and synchronous subqueue construction does not involve ``branch`` at all and produces a queue as efficient as if you had copy-pasted the queue segments directly into your code at that point. For instance, creating a reusable subqueue for error logging that creates the log line, logs it to a console, to a logfile, and to [statsd](https://github.com/msiebuhr/node-statsd-client):

{% highlight js %}
function errorLogSubqueue(queueName, parentQ) {
    return parentQ
        .map(function(error) {
            return queueName + " " + error.stack.split('\n')[1] + " " + error.message + '\n';
        })
        .each(console.err)
        .each(fs.appendFile.bind(fs, '/var/log/mySystem/error.log'))
        .map(function(errString) {
            return "error." + queueName + "." + errString.replace(/^.*([A-Za-z0-9.]*):([0-9]*):[0-9]*.*$/, "$1.$2");
        })
        .each(sdc.increment);
};

q('someQueue')
    ...

q('someQueueError')
    .subQueue(errorLogSubqueue.bind(this, 'someQueue'));

q('someOtherQueue')
    ...

q('someOtherQueueError')
    .subQueue(errorLogSubqueue.bind(this, 'someOtherQueue'));
{% endhighlight %}

With subqueues, it should now be possible for queue-flow-based data processing libraries to be easily implementable and shareable (such as streaming words from a text into a queue to analyze frequency and generate a simple search engine over the documents, for instance).

## parallelism

queue-flow recently underwent a fairly extensive rewrite of how its data processing methods behave, passing the control of when processed data is sent down to the next queue to the constructor function rather than in each of the data processing methods. The queue-flow constructor function simply passes along the result as soon as it is informed by the data processing method.

So why do this? Because queue-flow allows the constructor function to be overridden by another constructor function. Until recently, only [``sloppy-queue-flow``](https://github.com/dfellis/sloppy-queue-flow) was the only alternate constructor function (that I'm aware of), and was specifically designed to allow queue-flow-style code organization but not worry about maintaining proper queueing order (useful for handling request-response style code where any particular request and response has nothing to do with any other).

But a few days ago, a working version of [``parallel-queue-flow``](https://github.com/dfellis/parallel-queue-flow) was released based on this ability for the constructor function to determine when a result is passed along. Simply, parallel-queue-flow will fire off several concurrent requests (up to the concurrency-level specified, or simply all enqueued values if not specified) and keep track of the original order of these values.

If there are three values enqueued, and the second value is returned first, parallel-queue-flow will hold onto it until the first value is returned, and then pass both values down in the correct order. Then when the third value arrives it will be passed down immediately since the first two have been passed along already.

For most code, all you need to do is change this:

{% highlight js %}
q('someQueue')
    .map(function(val) {
        return val*Math.sin(val);
    })
    .map(someRemoteApiCall)
    ...
{% endhighlight %}

to this:

{% highlight js %}
q('someQueue', parallel(os.cpus().length))
    .map(function(val) {
        return val*Math.sin(val);
    })
    .map(someRemoteApiCall)
    ...
{% endhighlight %}

The algorithm takes advantage of Javascript's treatment of object assignment as a [pass-by-reference](http://publib.boulder.ibm.com/infocenter/lnxpcomp/v8v101/index.jsp?topic=%2Fcom.ibm.xlcpp8l.doc%2Flanguage%2Fref%2Fcplr233.htm) so a closure and an array can both hold the exact same object in memory (so changes in one will show up in the other), and the fact that now the actual passing of the value to the next queue is done via a closure that can be passed around. You can read the source of parallel-queue-flow's ``fire`` and ``handlerCallback`` methods to see the algorithm implementation. I may write up another blog post explaining the algorithm if interest is expressed.

The ``parallel-queue-flow`` parallel algorithm is somewhat naive, so it cannot guarantee the accuracy of results from ``reduce`` or ``reduce``-derived methods like ``toArray`` (but is written such that if they are synchronous methods they will work, so ``toArray`` *should* work). If the reducer method is asynchronous and the output variable is not an object, it is possible for values to be lost during the reduction. It's best to ``branch`` at this point into a standard queue-flow queue from the ``parallel-queue-flow`` queue to be sure the reduction occurs correctly.

It is possible to create a functioning parallel algorithm for ``reduce``-type operations, but it will slow down the parallelism of simpler, more commonly-used operations such as ``map`` and ``filter``. On top of that, it can only guarantee that the ``reduce`` operation will work if the ``reducer`` is [associative](http://en.wikipedia.org/wiki/Associative_property) or [commutative](http://en.wikipedia.org/wiki/Commutative_property) and the initial value can be applied multiple times without affecting the outcome (that the initial value is an [identity element](http://en.wikipedia.org/wiki/Identity_%28mathematics%29#Identity_element)).

Since ``reduce``-style operations coincide closely with discrete mathematics, I recommend sticking with the regular queue-flow constructor function unless you're sure that you can use one of the other constructor functions that provide fewer guarantees on whether your arbitrary code will function correctly (for instance, ``sloppy-queue-flow`` only guarantees synchronous, commutative reducers, so not even ``toArray`` is guaranteed to work as expected).

## promises

A large segment of the Javascript community supports a pattern called ["promises", which have been standardized by CommonJS](http://wiki.commonjs.org/wiki/Promises/A). Promises are used by several Node.js libraries as well as [jQuery, which calls them Deferred Objects](http://api.jquery.com/category/deferred-object/).

Promises are a solution to asynchronous value handling, and queue-flow's new ``promise`` method makes working with promises simpler. When constructing your queue, you pass it the promise constructor function. Each value passed into the queue constructs a new promise using that value or array of values as the argument or arguments for creating the promise. ``queue-flow`` registers the handlers for the result or error, passing the result into the next queue and the error into the named error queue, just like the ``node`` method does for Node.js-style callback functions, meaning ``promise`` promises to be just as significant for the browser with jQuery as ``node`` is for Node.js' native API.

{% highlight js %}
q(['http://someurl.com/some/path'])
    .map(function(url) {
        return {
            url: url,
            dataType: 'json',
            timeout: 3000
        };
    })
    .promise($.ajax)
    .map(JSON.parse)
    .each(function(json) {
        $('#' + json.id + ' >  #name').text(json.name);
        ...
    });
{% endhighlight %}

## Future developments

``queue-flow`` should hopefully reach a stable point again, noted by a 0.6.0 release, in a month or two. There is a rough roadmap, which I will discuss below, to reach 0.6.0, but more than half of the new features in the 0.5.x releases were unplanned, added as use-cases confronted me while dogfooding the library. This means if there's a feature you'd like to see that isn't there, yet, you're more significantly more likely to see it if you ask about it. :)

The plans for ``queue-flow`` can be roughly divided into short, medium, and long term plans, where short will definitely happen in the next month or two, medium term should be doable in months 3-6 after that, but may be pushed back or pulled forward as the priorities at that time warrant, and long term are basically a written record that I *want* to do this, and hope to do it some day, and if people agree that its a good idea I'll pull it into the medium term with a rough timeline to do it. :)

Short term, I intend to start profiling code paths to make sure I'm using the faster way to do things rather than the more easily maintainable (though I trust my slowly developed judgment on software quality that I haven't shot myself in the foot with any particular algorithm).

I also want to solidify the methods of ``queue-flow``; that is, decide with 95% confidence that I have all of the methods you'd want to create queues that are readable and can interface easily with 80% of all JS libraries out there. I have no idea how to quantify "95% confidence" right now, though.

Finally, I want to beef up the tests for ``sloppy-queue-flow`` and ``parallel-queue-flow`` so they cover everything I *think* they should theoretically be able to cover (and therefore discover the deficiencies if they exist) so these two constructor functions are just as much "first-class" as ``queue-flow``.

Medium term, I plan on building upon ``parallel-queue-flow`` to produce a few derivative alternate constructor functions. Some ideas include ``thread-queue-flow`` based on [webworker-threads](https://github.com/audreyt/node-webworker-threads), ``cluster-queue-flow`` based on [Node.js's cluster API](http://nodejs.org/api/cluster.html), and ``auto-queue-flow`` that dynamically alters the parallelism of any step in the queue to try keep the queue throughput at a specified rate of items processed per second.

I also like the back-and-forth inspiration between me and [jprichardson](https://github.com/jprichardson), and would like to figure out a way that errors in the queue could be monitored by event handlers as in his [node-qflow](https://github.com/jprichardson/node-qflow) library (I don't know how to do that yet without specifying a ridiculous number of event handlers with my design.)

Long term, I have some more esoteric alternate constructor functions that I would like to write, such as an ``autothread-queue-flow`` and ``autocluster-queue-flow`` that merges the ``auto-queue-flow`` concept with the ``thread-`` and ``cluster-queue-flow`` concepts, and then expand that into an ``aws-queue-flow`` that automatically allocates new AWS instances in anticipation of increasing CPU demand to push queue processing onto, or alternately destroys as the load decreases.

That's not too ambitious, right?
