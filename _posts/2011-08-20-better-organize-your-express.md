---
layout: site
title: Better organize your Express web application with express-route-util
subtitle: 
---
So, you've chosen to use [Node.js](http://nodejs.org/) for your next project, perhaps because of the [various](http://attractivechaos.github.com/plb/) [benchmarks](http://onlyjob.blogspot.com/2011/03/perl5-python-ruby-php-c-c-lua-tcl.html) showing the V8 Javascript engine getting a dynamically-typed language to perform as well as statically-typed languages, and Node.js's HTTP performance [surpassing](http://zgadzaj.com/benchmarking-nodejs-basic-performance-tests-against-apache-php) [Apache's](http://www.synchrosinteractive.com/blog/9-nodejs/22-nodejs-has-a-bright-future).
 
But, your web application isn't doing anything special with the HTTP protocol, and you don't want to [handle that manually](http://nodejs.org/docs/v0.4.11/api/http.html), so you chose to use the [Express framework](http://expressjs.com/) to take care of that.
 
So you start out using Express similarly to MVC frameworks like [Ruby on Rails](http://rubyonrails.org/) or [Django](https://www.djangoproject.com/), but there are a few things that are missing. Some of that seems fine; there's no explicitly-chosen Model, but that means you can choose your own, like [Mongoose](http://mongoosejs.com/), [FastLegS](https://github.com/didit-tech/FastLegS), or [RDX](https://github.com/kreetitech/RDX). Express also supports a few different view template engines such as [jQuery Templates](https://github.com/kof/node-jqtpl), [Jade](https://github.com/visionmedia/jade), and [EJS](https://github.com/visionmedia/ejs).
 
And controllers seem to be the easiest portion of Express, just write:

{% highlight js %}
app.get("/path/to/page", function(request, response) {
    response.render("myView", {hello: "world"});
}
{% endhighlight %}
        
But some problems arise. The controller functions grow huge with anonymous callbacks as you interact with your model, there's copy-pasted code in several of them as you include session handling for login verification, and your source file is huge as these controllers are all in the same scope as the express library.
        
Express has solutions to all of these problems. The method calls to register your controller can actually accept any number of functions, any one of which can choose to render an output for the user, or "pass the buck" along to the next controller in the list (by calling "next"). So common functionality can be abstracted, like so:

{% highlight js %}
function isLoggedIn(request, response, next) {
    if(request.session.loggedIn) {
        next();
    } else {
        response.redirect("/loginRequired");
    }
}

app.get("/path/to/page", isLoggedIn, function(request, response) {
    response.render("myView", {hello: "world"});
}
{% endhighlight %}
            
Now your functions have been reduced in size, do just one thing each (perhaps piggybacking model data on the request object) and it *feels* better. But there's still a few problems: it's all in the same source file so its hard to find exactly what you're looking for, it's hard to get a bigger picture about what it is your web application is doing, and it's hard to make a change to any controller's URL because references to it are strewn about everywhere.
            
The [express-route-util](https://github.com/AGROSICA/express-route-util) was written with this in mind. Unlike Ruby on Rails or [express-resource](https://github.com/visionmedia/express-resource) which solve this problem by restricting the meaning of controllers to directly correspond to a URL (and changing a URL requires changing the controller name), and restricts what controller names are valid, this utility continues to expose the full functionality of the Express engine while helping you organize your source code and URLs.
            
The controllers are defined to use a directory hierarchy like so:
            
    controllers/
        controllers.js
        social/
            controllers.js
            helper_lib.js
            
The utility only imports the export objects of the ``controllers.js`` files, so other code not to be used as a controller can reside within separate library files for even better source organization.
            
The exported functions are referenced relative to their location in the directory hierarchy, using a dot-notation just like a Javascript object hierarchy, so exported functions from the controllers.js in the root of the directory structure are at the root of the hierarchy, such as "index", "login", and "logout", while those in the social directory's controller.js are referenced by "social.viewProfile", etc.
            
The URL hierarchy is defined by the user in a simple JSON tree such as:

{% highlight js %}
{
    '/': 'index',
    'get,post/login': 'login',
    'post/logout': ['common.requireLogin', 'logout'],
    '/social' : {
        '/': 'social.index',
        '/profile': {
            '/:username': 'social.viewProfile',
            'get,post/edit': ['common.requireLogin', 'social.editProfile']
        }
    }
}
{% endhighlight %}

The basic syntax for the keys is ``[HTTP methods]/url/path/:variable``, where ``[HTTP methods]`` is an optional, case-insenstive comma-separated list of HTTP methods, and the rest of the key is a standard Express URL.

The value for a key can be a string indicating the particular controller to use, an array of strings indicating the controllers to use and the order to use them, or an object, following these same patterns, allowing a sub-sectioning of URLs.

When a key is holding an object, any HTTP methods you indicate are ignored, and the given URL is prepended to the URLs of all keys within the object, so the ``social.editProfile`` controller would have a URL of ``/social/profile/edit/`` and the URL will function for both GET and POST requests.

The intended usage of express-route-util is as follows:

{% highlight js %}
var router = require("express-route-util");

router.setDefaultMethod("get");
router.registerRoutes(app, urlJSON, "/path/to/controllers/");
var href = router.getControllerUrl("social.viewProfile", { username: "dellis" });
{% endhighlight %}

Where ``app`` is the Express application object, ``urlJSON`` is the JSON object relating paths to controllers, and ``"/path/to/controllers/"`` is the location of the controllers for the project (can be a relative URL).

A "good" set of defaults for registerRoutes is:

{% highlight js %}
router.registerRoutes(app, JSON.parse(fs.readFileSync("./paths.json")), "./controllers");
{% endhighlight %}

This way, you have an overall hierarchy similar to:

    app/
        controllers/
            controllers.js
            ...
        public/
            ...
        views/
            ...
        app.js
        paths.json

And you'll probably want to attach the ``router`` to the ``global`` object, as the ``getControllerUrl`` is most useful to the controllers to generate the URLs to pass to the views.

To install using [NPM](http://www.npmjs.org/), type:

    npm install express-route-util

That's all there is to it!