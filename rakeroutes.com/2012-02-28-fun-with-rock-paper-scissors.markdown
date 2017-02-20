---
layout: single
title: "Fun with Rock, Paper, Scissors"
date: 2012-02-28 11:23
comments: true
categories: ruby
---

[James Edward Gray II's Ruby Quiz #16](http://www.rubyquiz.com/quiz16.html) was
to implement Rock, Paper, Scissors playing classes to compete on a playing
field managed by a given Game class. Today we revisit this quiz for a bit of
coding fun. We'll implement some simple players and move on to some basic
metaprogramming techniques and write players that manipulate each other or the
game itself.

<!-- more -->

If you want to follow along:

```
% git clone git://github.com/sdball/ruby_quiz.git
% cd ruby_quiz/16
# ruby -I . rock_paper_scissors.rb [player files to run]
% ruby -I . rock_paper_scissors.rb players/
% ruby -I . rock_paper_scissors.rb players/always_rock.rb players/always_scissors.rb
AlwaysRock vs. AlwaysScissors
  AlwaysRock: 1000
  AlwaysScissors: 0
  AlwaysRock Wins
```

## Simple Player

Right. So here's what a very simple player class looks like.

```ruby
# Michael Bluth
class AlwaysRock < Player
  def choose
    :rock
  end
end
```

The `rock_paper_scissors.rb` defines a `Player` class, a `Game` class, and
provides a script to run the contest. Our own custom player classes just have to
inherit from the given Player class and implement a `choose` method that returns
`:rock`, `:paper`, or `:scissors`. Our players can optionally implement a
`result` method that is used as a callback from the game to allow our players to
see the result of their play. If our players have an `initialize` method it is
called with the classname of the opponent.

So, back to Michael Bluth (or Poor Predictable Bart) who always chooses rock.
His `choose` method just returns the symbol `:rock`. Easy. Let's write a player
to beat him. Also easy.

```ruby
# GOB
class AlwaysPaper < Player
  def choose
    :paper
  end
end
```

GOB will beat Michael Bluth 100% of the time.

```
% ruby -I . rock_paper_scissors.rb players/always_rock.rb players/always_paper.rb
AlwaysRock vs. AlwaysPaper
  AlwaysRock: 0
  AlwaysPaper: 1000
  AlwaysPaper Wins
```

Ok, now let's write a player to beat them both. Consistently.

## Reactive Player

```ruby
class Reactive < Player
  MOVE_THAT_BEATS = {
    rock: :paper,
    paper: :scissors,
    scissors: :rock
  }

  def choose
    @my_move || :paper
  end

  def result(my_move, opponent_move, outcome)
    @my_move = MOVE_THAT_BEATS[opponent_move]
  end
end
```

This player is kind of interesting, but still straightforward. Reactive has a
constant `MOVE_THAT_BEATS` that maps moves to the move that would beat them.
Reactive then uses that knowledge to play the move that beats the last move
played by his opponent (or :paper). This strategy should prove extremely
effective against a player who always makes the same move.

```
% ruby -I . rock_paper_scissors.rb players/always_rock.rb players/always_paper.rb players/reactive.rb
AlwaysRock vs. Reactive
  AlwaysRock: 0
  Reactive: 1000
  Reactive Wins
AlwaysPaper vs. Reactive
  AlwaysPaper: 0.5
  Reactive: 999.5
  Reactive Wins
```

Yep, Always Rock loses every game because Reactive plays paper from the start. Paper
gets in one draw, then loses every game afterwards.

Now, let's write a player that will beat all of the players thus far.

## Random Player

Theoretically, the best strategy in Rock, Paper, Scissors is to be completely
random. That's a pretty easy strategy for a computer to play (although
surprisingly difficult for a human) so let's code that up next.

```ruby
class RandomPlayer < Player
  def choose
    [:rock, :paper, :scissors].shuffle.first
  end
end
```

Well that was easy. Let's pit Random against some of the players so far.

```
% ruby -I . rock_paper_scissors.rb players/always_rock.rb players/random.rb
# out of 10 runs, Random won 8 games
```

```
% ruby -I . rock_paper_scissors.rb players/reactive.rb players/random.rb
# out of 10 runs, Random won 8 games
```

Fun, but not all that interesting. How about we write a player who watches their
opponent and builds a strategy to defeat them? This player is going to be a bit
more complex than those we've written so far.

## Pattern Matching Player

The plan: build up a pattern memory of player moves, opponent moves, and the
next move the opponent played under those conditions. After every move:

- remember the moves that were just played
- record the *previous* game's moves and the opponents move
- use that record of game patterns to determine the opponent's next likely move
  and play the move that beats it

This pattern set will allow us to make observations such as "the last ten times
I played rock and my opponent played paper, my opponent's next move was
scissors" or "out the last 97 times I played rock and my opponent played
scissors my opponent played scissors next 96% of the time".

```ruby
class PatternMatching < Player
  MOVE_THAT_BEATS = {
    rock: :paper,
    paper: :scissors,
    scissors: :rock
  }
  def initialize(opponent)
    @first_game = true
    @patterns = {
      rock: {},
      paper: {},
      scissors: {}
    }
  end

  def choose
    @my_move || :rock
  end

  def result(mine, theirs, outcome)
    store_moves(mine, theirs)
    plan_next_move
    @first_game = false
  end

  private
  def store_moves(mine, theirs)
    unless @first_game
      store_pattern(mine, theirs)
    end
    @my_last_move = mine
    @their_last_move = theirs
  end

  def store_pattern(mine, theirs)
    if @patterns[@my_last_move][@their_last_move]
      @patterns[@my_last_move][@their_last_move] << theirs
    else
      @patterns[@my_last_move][@their_last_move] = [theirs]
    end
  end

  def plan_next_move
    @my_move = MOVE_THAT_BEATS[their_likely_next_move] || :rock
  end

  def their_likely_next_move
    their_moves = @patterns[@my_last_move][@their_last_move]
    return [:rock, :paper, :scissors].shuffle.first if their_moves.nil?
    count = Hash.new(0)
    their_moves.each do |move|
      count[move] += 1
    end
    count.sort_by {|key, value| value}.last.first
  end
end
```

This guy looks complicated, but the only tricky logic is around the whole "Is
this the first game or not?" I'm not entirely happy with the way I've worked it
out here, but it gets the job done. `store_pattern` and `their_likely_next_move`
are the two key methods. The move patterns are stored as nested hashes `[my
move][opponent move] => [array of observed moves]` e.g. `[:rock][:paper] =>
[:scissors, :paper, :scissors, :scissors, :scissors, :paper]

`their_likely_next_move` just looks into the pattern and counts up the data seen
so far. If there's no data yet, it just guesses randomly.

Let's see how our pattern matcher fares!

```
% ruby -I . rock_paper_scissors.rb players/always_rock.rb players/pattern_matching.rb
AlwaysRock vs. PatternMatching
  AlwaysRock: 1.0
  PatternMatching: 999.0
  PatternMatching Wins
```

Not bad! Our pattern matcher correctly determines the simple pattern employed by
Always Rock

```
% ruby -I . rock_paper_scissors.rb players/reactive.rb players/pattern_matching.rb
Reactive vs. PatternMatching
  Reactive: 3.5
  PatternMatching: 996.5
  PatternMatching Wins
```

Bam! Our pattern matcher handily figures out what our simple Reactive player is
likely to do. Now for a real challenge.

```
% ruby -I . rock_paper_scissors.rb players/random.rb players/pattern_matching.rb
RandomPlayer vs. PatternMatching
  RandomPlayer: 491.0
  PatternMatching: 509.0
  PatternMatching Wins

% ruby -I . rock_paper_scissors.rb players/random.rb players/pattern_matching.rb
RandomPlayer vs. PatternMatching
  RandomPlayer: 518.5
  PatternMatching: 481.5
  RandomPlayer Wins
# it goes back and forth
```

So PatternMatching fares well against RandomPlayer but can't consistently win.
Let's make a player who can consistently and completely defeat RandomPlayer.
Let's make a player who *cheats*.

## Cheater

```ruby
class Cheater < Player
  def initialize(opponent)
    Kernel.const_get(opponent).class_eval('def choose; :scissors; end')
  end

  def choose
    :rock
  end
end
```

Mwahaha! Cheater utilizes that so far unused ability to get the name of his
opponent. Using that and some magic Cheater hypnotizes his opponent into always
playing scissors.

Does it work? Absolutely.

```
% ruby -I . rock_paper_scissors.rb players/always_rock.rb players/cheater.rb
AlwaysRock vs. Cheater
  AlwaysRock: 0
  Cheater: 1000
  Cheater Wins

% ruby -I . rock_paper_scissors.rb players/reactive.rb players/cheater.rb
Reactive vs. Cheater
  Reactive: 0
  Cheater: 1000
  Cheater Wins

% ruby -I . rock_paper_scissors.rb players/pattern_matching.rb players/cheater.rb
PatternMatching vs. Cheater
  PatternMatching: 0
  Cheater: 1000
  Cheater Wins

% ruby -I . rock_paper_scissors.rb players/random.rb players/cheater.rb
RandomPlayer vs. Cheater
  RandomPlayer: 0
  Cheater: 1000
  Cheater Wins
```

So what's going on here? To answer that, let's dive into `irb`

``` ruby
% irb -I .
1.9.3p0 :001 > require 'rock_paper_scissors'
 => true
1.9.3p0 :002 > require 'players/always_rock'
 => true
```

Ok, we're in irb and we've got our game and a player loaded. Let's see what Cheater is up to.

``` ruby
Kernel.const_get(opponent).class_eval('def choose; :scissors; end')
```

``` ruby
1.9.3p0 :003 > Kernel.const_get('AlwaysRock')
 => AlwaysRock
1.9.3p0 :004 > Kernel.const_get('AlwaysRock').class
 => Class
```

`const_get` checks a module for a constant with the given name. Since a class is
defined as a constant and Kernel sits over every class, we can ask Kernel to
find our opponent's class for us. Which is what we're doing here.

Basically `Kernel.const_get('AlwaysRock')` is an easy way to turn the string
"AlwaysRock" into a constant.

Now `class_eval`. This method just says to a class, "evaluate this string as if
it were written as part of your code."

```ruby
1.9.3p0 :005 > String.class_eval('def magic; "magic!"; end')
 => nil
1.9.3p0 :006 > "hi".magic
 => "magic!"
```

Put those pieces together, and our Cheater:

1. Takes the string of his opponents name and turns it into a constant to get at his opponent's class.
2. Rewrites his opponent with a new `choose` method that he controls.
3. Beats his opponents modified choose method.

```ruby
1.9.3p0 :007 > AlwaysRock.new('Cheater').choose
 => :rock
1.9.3p0 :008 > Kernel.const_get('AlwaysRock').class_eval('def choose; :scissors; end')
=> nil
1.9.3p0 :009 > AlwaysRock.new('Cheater').choose
=> :scissors
```

Insidious! Even worse, if you're running a contest with more than two players
the cheater permanently modifies his opponents into always playing their
redefined choose method.

Let's level the playing field and force the cheater to play fair.

## Level Playing Field

LevelPlayingField avoids being manipulated by the cheater by having a strong
will to resist Cheater's hypnotism. Cheater works by modifying the class of his
opponents, so LevelPlayingField doesn't have a choose method defined by his
class. LevelPlayingField defines his own choose method on initialization.

```ruby
class LevelPlayingField < Player
  def initialize(opponent)
    # prevent cheater from winning by waiting to pick a strategy
    self.class.class_eval do
      def choose
        [:rock, :paper, :scissors].shuffle.first
      end
    end
  end
end
```

Yes, that's right. LevelPlayingField uses the same trick that Cheater does, but
on himself to try and guarantee he's playing his own strategy.

```ruby
1.9.3p0 :010 > require 'players/level_playing_field'
 => true
1.9.3p0 :011 > LevelPlayingField.new('Cheater').choose
=> :paper
1.9.3p0 :012 > Kernel.const_get('LevelPlayingField').class_eval('def choose; :scissors; end')
=> nil
1.9.3p0 :011 > LevelPlayingField.new('Cheater').choose
=> :rock # LevelPlayingField is random so you actually might see :scissors here
```

```
% ruby -I . rock_paper_scissors.rb players/cheater.rb players/level_playing_field.rb
Cheater vs. LevelPlayingField
  Cheater: 496.5
  LevelPlayingField: 503.5
  LevelPlayingField Wins
```

Now this works, but there's one flaw. Our LevelPlayingField player only succeeds
when he's initialized *after* the Cheater.

```
% ruby -I . rock_paper_scissors.rb players/level_playing_field.rb players/cheater.rb
LevelPlayingField vs. Cheater
  LevelPlayingField: 0
  Cheater: 1000
  Cheater Wins
```

If Cheater comes into play after LevelPlayingField has completed his trick to
try and isolate his choose method, then the Cheater will still just override
LevelPlayingField's choose method. It's a remarkably tough strategy to defeat.
Any trick we use to cleverly hide our choose method from the Cheater can
ultimately be sidestepped by the Cheater just supplying a new `choose` method.

Even if we write a player who removes the ability for the Cheater to cheat, if
he's loaded first: he wins.

```ruby
class LevelPlayingField < Player
  def initialize(opponent)
    # rewrite class_eval to prevent cheating
    Module.class_eval('def class_eval(obj) end')
  end
  def choose
    [:rock, :paper, :scissors].shuffle.first
  end
end
```

So let's bring in a player who always wins. A player who out-cheats the cheater.
A player who modifies the game itself.

## Batman

```ruby
class Batman < Player
  def initialize(opponent)
    Game.class_eval do
      def play( match )
        match.times do
          next win @player1, :rock, :scissors if @player1.instance_of? Batman
          next win @player2, :rock, :scissors if @player2.instance_of? Batman
        end
      end
    end
  end
end
```

Batman's playing strategy is to enter the Game class itself and ensure that he's
the winner no matter what's been played. Not only that, but he completely alters
the game for all other player matches. They all get recorded as draws, since
only Batman can win.

```
% ruby -I . rock_paper_scissors.rb players/cheater.rb players/batman.rb
Cheater vs. Batman
  Cheater: 0
  Batman: 1000
  Batman Wins

% ruby -I . rock_paper_scissors.rb players/batman.rb players/cheater.rb
Batman vs. Cheater
  Batman: 1000
  Cheater: 0
  Batman Wins

% ruby -I . rock_paper_scissors.rb players/batman.rb players/reactive.rb
Batman vs. Reactive
  Batman: 1000
  Reactive: 0
  Batman Wins
```

Gosh, makes you almost feel bad for his opponents.

I hope you've enjoyed this bit of fun. On Thursday I plan to take this small
glimpse into the world of metaprogramming a bit further.
