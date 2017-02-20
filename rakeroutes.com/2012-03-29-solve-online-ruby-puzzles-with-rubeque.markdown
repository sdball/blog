---
layout: single
title: "Solve online Ruby puzzles with Rubeque"
date: 2012-03-29 14:26
comments: true
categories: ruby
---

Ruby programmers no longer need to be envious of
[CodingBat](http://codingbat.com/)
providing quick, fun, online programming puzzles for Python and Java
programmers! The SciMed guys have launched a great Ruby puzzle website called
[Rubeque](http://rubeque.com/).

<!-- more -->

The puzzles are currently skewed toward easy, but you can submit your own
puzzles for inclusion (awesome) so I expect harder puzzles will start popping
up as the site grows. There's still plenty to do now with 91 puzzles to solve.

They start off nicely with
[The Truth](http://rubeque.com/problems/the-truth)

```ruby
assert_equal true, __
```

Which of course can be solved with:

```ruby
true
# or !false
# or !!0
# or 0.zero?
# etc.
```

The puzzles are all online, the site is gorgeous, and you can sign-in with your
github account. Complete win. After you've put in a successful puzzle entry you
get to see other solutions that people have come up with; I've already learned
some interesting techniques like passing an initial argument to inject

```ruby
[].inject(0, :+)
# >> 0
[1].inject(0, :+)
# >> 1
```

There even already a couple of interesting meta-programming puzzles:

- [Method Acting](http://rubeque.com/problems/method-acting)
- [Keep Our Parks Clean!](http://rubeque.com/problems/keep-our-parks-clean-excl-)

So get on there and have fun!
