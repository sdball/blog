---
layout: single
title: "The problem with completely client-side applications"
date: 2012-03-27 12:08
comments: true
categories: javascript
---

I've been spending a lot of time on an application where almost all of the logic
is in the client. It's been a fascinating tour of the bleeding edge of
event-driven modular JavaScript code. Here's what I've learned.

<!-- more -->

## Conventions for application composition don't exist

Should you use require.js to build up modular application? Should you use
Backbone.Marionette to compose your application? Should you use JST templates?
Underscore templates? Mustache? Underscore with mustache syntax?

## Conventions for testing don't exist

Even the generally awesome
[Backbone Boilerplate](https://github.com/tbranyen/backbone-boilerplate)
which comes with *two* testing libraries for you to pick from doesn't have tests
that are actually integrated into the application. The tests simply demonstrate
Jasmine's rspec syntax vs QUnit's assertion syntax.

Look around the web for Backbone tutorials. There are a few that are really
great. Now find *one* with a full application that demonstrates good testing.

This gets even more complicated due to the lack of conventions for building the
app. Testing a backbone application written using require.js is very different
from testing one that doesn't.

## Conventions for deployment don't exist

So you've got a working application that may or may not be well tested. How are
you going to deploy it? Since there's no conventional way to construct the
application, there's no conventional way to deploy it.

App construction solution A might work really well for your team and be well
documented for development, but then lack docs or examples for actual
deployment.

App construction solution B might be really well documented for deployment, but
be way too simplistic a construction method to build your app in.

## Yet?

Maybe the lack of these conventions is a growing pain and the price you pay for
being on the cutting edge. But these conventions aren't a new idea. At the very
least testing should be built in at a fundamental level. The fact that *testing*
isn't addressed is a big red flag. Either the community has decided that testing
isn't really that important (yikes) or each dev group is coming up with their
own solution for testing and not sharing that back out (yikes).

Either way, this is exactly where Rails' strengths can build up an application.
Rails comes with its own conventions about how web applications should be
constructed, where files should go, how the code should be deployed, and how
code should be tested. Rails makes testing its own API responses completely easy
(or at least simple) and provides hooks for testing the JavaScript UI.
