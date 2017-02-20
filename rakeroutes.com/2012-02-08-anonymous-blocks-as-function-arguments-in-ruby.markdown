---
layout: single
title: "Anonymous blocks as function arguments in Ruby"
date: 2012-02-08 19:49
comments: true
categories: [ruby, programming]
---

A quick tidbit that comes in handy when you want to write some idiomatic
Ruby: how to write methods that accept optional blocks of code to execute.

<!-- more -->

Any Ruby programmer should be familiar with using blocks to scope code execution.

``` ruby
[1,2,3].each { |n| puts n }
# >> 1
# >> 2
# >> 3
```

``` ruby
# whee!
100.times do
  puts 'Hello, World!'
end
```

Easy right? In these cases `each` and `times` are (essentially) Ruby methods
that accept a block as a parameter. Very handy, very easy to read, very clear.
And something we can use in our own code.

Any Ruby method can accept a block. In fact any Ruby method *does* accept a
block, it will just silently ignore it unless it's told to do something with it.

``` ruby
# follow along in irb!
def eat(meal) "delicious!" end
puts eat(['cheese'])
# => 'delicious!'
puts eat(['cheese', 'steak', 'wine']) { |food| "mmm #{food}" }
# => 'delicious!'
```

Whoa, see that? Our method just happily said, "Yeah, ok. Block parameter. Got
it. Not going to even care because I don't know what to do with it." As you'll
see, the block is actually being passed into the method; it's just being passed
in secretly. This is great because it means that you have to do very little work
to get any method to accept and process a block, but it can be a little
surprising until you get used to it.

Let's jump into TextMate (or your favorite editor) and put a bit more into this
method. Let's get it to actually execute our passed block of code for every item
of food in the meal. There are two ways to execute a block passed to your
function: `.call` and `yield`.

## yield

``` ruby
def eat(meal)
  meal.each {|food| yield(food)}
  'delicious!'
end

puts eat(['cheese', 'steak', 'wine']) {|food| puts "Mmm, #{food}"}
# >> Mmm, cheese
# >> Mmm, steak
# >> Mmm, wine
# >> delicious!

puts eat(['cheese'])
# ~> -:2:in `block in eat': no block given (yield) (LocalJumpError)
```

Aww. Now we've got our function to call out to a passed block, but it isn't
optional. Well, we can fix that right up.

``` ruby
def eat(meal)
  if block_given?
    meal.each {|food| yield(food)}
  end
  'delicious!'
end

puts eat(['cheese', 'steak', 'wine']) {|food| puts "Mmm, #{food}"}
# >> Mmm, cheese
# >> Mmm, steak
# >> Mmm, wine
# >> delicious!

puts eat(['cheese'])
# >> delicious!
```

Hooray! Now we have a cool little method that can expand its horizons if we give
it a block of code.

## .call

Another way to process the passed block is to explicitly list it in
the method arguments. The trick here is that a block argument has to be last,
and has to be prepended with an ampersand (&amp;). The &amp; is a bit of magic
that converts the anonymous block into a Proc that we can directly reference.
Here's how that works.

``` ruby
def eat(meal, &consume)
  if block_given?
    meal.each {|food| consume.call(food)}
  end
  'delicious!'
end

puts eat(['cheese', 'steak', 'wine']) {|food| puts "Mmm, #{food}"}
# >> Mmm, cheese
# >> Mmm, steak
# >> Mmm, wine
# >> delicious!

puts eat(['cheese'])
# >> delicious!
```

Since we've named our block as an argument, we can analyze it directly for logic flow.

``` ruby
def eat(meal, &consume)
  if consume
    meal.each {|food| consume.call(food)}
  end
  'delicious!'
end

puts eat(['cheese', 'steak', 'wine']) {|food| puts "Mmm, #{food}"}
# >> Mmm, cheese
# >> Mmm, steak
# >> Mmm, wine
# >> delicious!

puts eat(['cheese'])
# >> delicious!
```

As you can see, we have a few options for how to write our method that accepts
an optional block. I don't know if there are legitimate reasons to prefer one or
the other, so choose whichever syntax you prefer.

## /record scratch/ Update from the community

**Feb 20th 2012**

Turns out: converting the passed block into a Proc is expensive. If all you are
doing with the Proc is calling it, you should consider switching that to the
`yield` syntax. But you do get something cool out of converting to a Proc
object: a Proc object. You can call it, store it, pass it somewhere else, or
introspect it for doing something clever.

For example, you can ask a Proc about its parameters.

``` ruby
def eat(meal, &consume)
  if consume
    # only call a block with parameters
    unless consume.parameters.empty?
      meal.each {|food| consume.call(food)}
    end
  end
  'delicious!'
end

puts eat(['cheese', 'steak', 'wine']) {|food| puts "Mmm, #{food}"}
# >> Mmm, cheese
# >> Mmm, steak
# >> Mmm, wine
# >> delicious!

puts eat(['cheese', 'steak', 'wine']) { puts "Mmm" }
# Nothing printed because our method spurns blocks without parameters.

puts eat(['cheese'])
# >> delicious!
```

Thanks to the awesome commenters Donald Ball, Ken Collins, and Cameron Desautels
for the info and insight.
