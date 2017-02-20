---
layout: single
title: "Coffee Taster: an Easy CoffeeScript Development Environment for Ruby Developers"
date: 2012-02-17 09:42
comments: true
categories: coffeescript
---

Announcing the first release of
[Coffee Taster](https://github.com/sdball/coffee-taster)!
An easy to install, easy to use CoffeeScript development environment for Ruby
developers.

<!-- more -->

## Getting started

First make sure you're using [RVM](https://rvm.beginrescueend.com/).

1. `git clone git://github.com/sdball/coffee-taster.git`
2. `cd coffee-taster`
3. `bundle install`
4. `rake watch`
5. Open index.html in a browser.

Hey-y-y-y! Check that out. Your browser just ran some CoffeeScript (that was
compiled into JavaScript when you ran the `rake watch` command).

It gets better!

Open `coffee/main.coffee`

> File: main.coffee

```coffeescript
$(document).ready ->
  $('ol').after('<h2>Hello from coffee-taster! (Added from coffee/main.coffee)</h2>')
```

Change it somehow.

> File: main.coffee

```coffeescript
$(document).ready ->
  $('ol').after('<h2>Whoa. This is awesome.</h2>')
```

And refresh your browser. Yeah!

No you'll be able to follow along with just about any basic CoffeeScript
tutorial, including those that manipulate the DOM or use jQuery. Like my
[jQuery Event Binding in CoffeeScript](http://coffeescriptcafe.com/blog/jquery-event-binding-in-coffeescript/) article on [CoffeeScript Cafe](http://coffeescriptcafe.com/).
