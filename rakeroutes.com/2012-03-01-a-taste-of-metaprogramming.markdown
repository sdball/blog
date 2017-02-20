---
layout: single
title: "A Taste of Metaprogramming"
date: 2012-03-01 14:13
comments: true
categories:
---

Today we take a small taste from the wide ranging metaprogramming abilities that
Ruby gives us. We'll be looking at `define_method` and `method_missing` to
take a class of hardcoded repetitive methods to class that has methods dynamically
created at runtime or methods that never really exist.

<!-- more -->

## Metaprogramming

First things first. In the fine tradition of the Ruby Rogues, let's lead with a
definition.

Metaprogramming is a loose term that simply describes the act of writing code
that itself writes code. We Rubyists generally use the term to describe code
that modifies itself or other code at runtime. For example, a class that defines
methods dynamically. Personally, I don't think of metaprogramming as all that
different from "regular" programming.

## The base class

```ruby
require "test/unit"

class Food
  def taste
    'Mmm.'
  end
  def smell
    'Smells good!'
  end
  def consume
    'Delicious!'
  end
end

class TestFood < Test::Unit::TestCase
  def setup
    @food = Food.new
  end
  def test_food_can_be_tasted
    assert_equal('Mmm.', @food.taste)
  end
  def test_food_can_be_smelled
    assert_equal('Smells good!', @food.smell)
  end
  def test_food_can_be_eaten
    assert_equal('Delicious!', @food.consume)
  end
end
```

```ruby
# Running tests:

...

Finished tests in 0.000499s, 6012.0240 tests/s, 6012.0240 assertions/s.

3 tests, 3 assertions, 0 failures, 0 errors, 0 skips
```

Ok, we've got working code with passing tests. We are now free to experiment!

The goal is to keep the exact same tests passing, but completely change the
class being tested.

## define_method

```ruby
require "test/unit"

module Eatable
  ACTIONS = {
    taste: 'Mmm.',
    smell: 'Smells good!',
    consume: 'Delicious!'
  }
end

class Food
  include Eatable
  ACTIONS.each_pair do |action, result|
    define_method(action) do
      result
    end
  end
end

class TestFood < Test::Unit::TestCase
  def setup
    @food = Food.new
  end
  def test_food_can_be_tasted
    assert_equal('Mmm.', @food.taste)
  end
  def test_food_can_be_smelled
    assert_equal('Smells good!', @food.smell)
  end
  def test_food_can_be_eaten
    assert_equal('Delicious!', @food.consume)
  end
end
```

```
# Running tests:

...

Finished tests in 0.000499s, 6012.0240 tests/s, 6012.0240 assertions/s.

3 tests, 3 assertions, 0 failures, 0 errors, 0 skips
```

Bam! Delicious contrived coding example. We have an Eatable module that defines
an `ACTIONS` constant of actions and results. The Food class then uses that
constant to dynamically create methods from it using `define_method`.

We could even use `ACTIONS` to drive our tests (although that's outside our goal.)

```ruby
require "test/unit"

module Eatable
  ACTIONS = {
    taste: 'Mmm.',
    smell: 'Smells good!',
    consume: 'Delicious!'
  }
end

class Food
  include Eatable
  ACTIONS.each_pair do |action, result|
    define_method(action.to_sym) do
      result
    end
  end
end

class TestFood < Test::Unit::TestCase
  def setup
    @food = Food.new
  end
  Eatable::ACTIONS.each_pair do |action, result|
    define_method(:"test_#{action}") do
      assert_equal(result, @food.send(action.to_sym))
    end
  end
end
```

```ruby
# Running tests:

...

Finished tests in 0.000498s, 6024.0964 tests/s, 6024.0964 assertions/s.

3 tests, 3 assertions, 0 failures, 0 errors, 0 skips
```

Wowzers. I think we've reached some kind of limit of contriving here. Our Food
and TestFood classes are barely distinguishable.

So, `define_method` allows us to write code that creates methods when it is
executed. Now we'll look at a way for our classes to have methods that *never
even existed*. We might as well call them Verbal Kints.

*\*crickets\**

From *The Usual Suspects*? Kevin Spacey's character through most of the movie?
He never really existed. So&hellip;

Ok, moving on.

## method_missing

`method_missing` is similar to define method, except it's called whenever a
class is called upon to execute a method that it doesn't already have defined.
`method_missing` takes the place of the called method, its return value is
returned just as if the method actually existed.

```ruby
require "test/unit"

module Eatable
  ACTIONS = {
    taste: 'Mmm.',
    smell: 'Smells good!',
    consume: 'Delicious!'
  }
end

class Food
  include Eatable
  def method_missing(method)
    ACTIONS[method]
  end
end

class TestFood < Test::Unit::TestCase
  def setup
    @food = Food.new
  end
  def test_food_can_be_tasted
    assert_equal('Mmm.', @food.taste)
  end
  def test_food_can_be_smelled
    assert_equal('Smells good!', @food.smell)
  end
  def test_food_can_be_eaten
    assert_equal('Delicious!', @food.consume)
  end
end
```

```ruby
# Running tests:

...

Finished tests in 0.000503s, 5964.2147 tests/s, 5964.2147 assertions/s.

3 tests, 3 assertions, 0 failures, 0 errors, 0 skips
```

What just happened? Well, `method_missing` takes at least one argument: the name
of the method that is trying to be called, which is passed in as a symbol. Since
the methods we're calling match up exactly with the `ACTIONS` hash, we can just
use that symbol as a key to return the result we want.

We Rubyists like to know if something quacks like a duck.

```ruby
Duck.new.respond_to? :quack
# >> true
```

When those methods don't ever actually exist (i.e. we're using `method_missing`
to pretend they are there), we can't ask an object if they have them!

```ruby
module Eatable
  ACTIONS = {
    taste: 'Mmm.',
    smell: 'Smells good!',
    consume: 'Delicious!'
  }
end

class Food
  include Eatable
  def taste
    ACTIONS[:taste]
  end
end

class Food_DM
  include Eatable
  ACTIONS.each_pair do |action, result|
    define_method(action.to_sym) do
      result
    end
  end
end

class Food_MM
  include Eatable
  def method_missing(method)
    ACTIONS[method]
  end
end

puts Food.new.respond_to? 'taste'
# >> true
puts Food_DM.new.respond_to? 'taste'
# >> true
puts Food_MM.new.respond_to? 'taste'
# >> false
# D'oh!

puts Food.new.cook
# throws NoMethodError
puts Food_DM.new.cook
# throws NoMethodError
puts Food_MM.new.cook
# silently fails! D'oh!
```

As you can infer, `method_missing` is a powerful tool that can result in
surprising code. We Rubyists don't like to be surprised either. If you really
need the power and flexibility of `method_missing` you should be prepared to go
the extra mile and keep your programs behaving as good Ruby citizens.

```ruby
class Food
  include Eatable

  def method_missing(method)
    if ACTIONS.has_key? method
      ACTIONS[method]
    else
      super(method)
    end
  end

  def respond_to?(method)
    ACTIONS.has_key? method || super(method)
  end
end

puts Food.new.respond_to? 'taste'
# >> true, woohoo!

puts Food.new.cook
# throws NoMethodError, woohoo!
```

There. Now we've got a Ruby program that acts as if it really has the methods
we're using; even though they don't exist.

I hope you've gotten at least a little out of this jaunt into Ruby
metaprogramming. If you'd like to learn more check out the pragprog book
[Metaprogramming Ruby](http://pragprog.com/book/ppmetr/metaprogramming-ruby) by
Paolo Perrotta.
