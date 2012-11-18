---
layout: site
title: node-localize, a Node.js localization library -- moving beyond gettext emulation
subtitle:
---
[node-localize](https://github.com/AGROSICA/node-localize) is a library that I've been working on for some time, and now it has enough features that I think it could be useful for others.

It's a localization library for Node.js, inspired by [GNU gettext](http://www.gnu.org/s/gettext/), but it does not restrict itself to the C-centric API of gettext, like a [couple](https://github.com/andris9/node-gettext) [others](https://github.com/DanielBaulig/node-gettext) do. (That in itself speaks about adapting gettext to Node.js. Should you use ``*.po`` files or ``*.mo`` fles? Does compiling the translations into a binary format designed for fast lookups make any sense for Node.js? Are the gains in space important?)
 
But this isn't about gettext, this is about node-localize, a library designed to work well with Node.js.

So, what advantages do we get over the gettext clones?

First, the library is an object in which multiple instantiations are possible, so you can handle your translations as you want; spawn a singleton object if you only need a global translation method, or spawn multiple if you want segmentation of the translations, such as static translations versus dynamically generated (user submitted?) translations.
 
Translations are stored in a simple JSON object and can be constructed in your application and loaded, or can be stored in a JSON file whose path is passed to the Localize object on instantiation, so they fit better in your workflow and can be manipulated programmatically with ease.
 
Translations can also insert variables into desired locations in the translated text with a syntax mixing jQuery and sprintf, which goes beyond gettext and allows the order of the inserted variables to differ between languages (particularly good for translating text between Asian and European languages).
 
Large translations can be stored in a special *translations* directory and referenced by a simple variable name (the variable name matching the file name of the default translation, minus the file extension).
 
node-localize also incorporates (and modifies, so it's forked) the [node-dateformat](https://github.com/felixge/node-dateformat) library so you can easily define date format types, translate them between languages, and use them on dates in your code.
 
The *xlocalize* command works in a way similar to *xgettext*, parsing source code and automatically generating translation templates for you (and correctly updating already generated translation files without overwriting your translations).
 
Planned features include pluralization support (the only feature of gettext not yet available in node-localize), numeral and currency formatting differences between languages, optional country code support (that falls back to the language baseline if not found), and support for running inside of browsers (possibly to include an exporting functionality to port desired translations to the browser, as well.
 
The entire API is viewable on the [github repository](https://github.com/AGROSICA/node-localize), and you can install it with a simple:

    npm install localize