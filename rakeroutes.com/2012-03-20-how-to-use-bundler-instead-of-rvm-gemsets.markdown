---
title: "How to use bundler instead of rvm gemsets"
date: 2012-03-20 11:42
---

Listening to the
[latest Ruby Rogues](http://rubyrogues.com/045-rr-bundler-with-andre-arko/) I
was intrigued to hear André Arko describe how using bundler can completely
obviate using rvm gemsets. He said that projects using bundler will just work if
you have all of your gems piled together. No need to go through the hassle of
managing a gemset for each project. I'm all about avoiding hassle, so I set out
to investigate.

<!-- more -->

Now, don't get me wrong: I love rvm. But I have to admit that managing gemsets
and running bundle install from scratch *again* for every new project is a pain.
I imagine that with this workflow I'll still use rvm to manage Ruby versions,
but not project gemsets.

## Install overlapping Rails versions into the same gemset

I picked Rails as the testbed for my conflicting version testing. I figure that
it's an easy gem to install, it brings in lots of versioned dependencies, and
it's extremely easy to see if things are working with a `rake test`.

I trusted that André wasn't leading me astray so I ran all these installs into
the 1.9.3 system gemset from rvm. If you want to follow along but be able to
easily clear out this collection of overlapping gems, just create a new gemset
for the experiment and delete it afterwards.

```
# Install multiple rails versions
% gem install rails # Rails 3.2.2
% gem install rails --version '~> 3.1.0'
% gem install rails --version '~> 3.0.0'
% gem list rails

*** LOCAL GEMS ***

rails (3.2.2, 3.1.4, 3.0.12)
```

There we go. Now to create a Rails project for each version.

## Aside: Running commands from a specific gem version

To run a command from a specific gem version, we need to use a cool trick that
the Ruby Rogues also mentioned on the Bundler show. To run a specific version of
a gem's binary you wrap the version number in underscores.

```
% rails -v
Rails 3.2.2

% rails _3.1.4_ -v
Rails 3.1.4

% rails _3.0.12_ -v
Rails 3.0.12
```

Pretty nifty eh? Sure if we were to have separate rvm gemsets for each project
then it wouldn't even be an issue; but I think this solution is quite
reasonable. Although I am entertaining the idea of writing a ZSH plugin that
will help run specific gem binaries if one doesn't already exist.

## Create a Rails project for each version

```
# create the projects
% rails new rails_3.2.2
% rails _3.1.4_ new rails_3.1.4
% rails _3.0.12_ new rails_3.0.12
```

## Scaffold and test each Rails project

```
% cd rails_3.2.2
% bundle install
% rails g scaffold post title:string author:string content:text posted_at:datetime
% rake db:migrate
% rake
Run options:

# Running tests:



Finished tests in 0.002877s, 0.0000 tests/s, 0.0000 assertions/s.

0 tests, 0 assertions, 0 failures, 0 errors, 0 skips
Run options:

# Running tests:

.......

Finished tests in 0.172671s, 40.5395 tests/s, 57.9136 assertions/s.

7 tests, 10 assertions, 0 failures, 0 errors, 0 skips
```

```
% cd rails_3.1.4
% bundle install
% rails g scaffold post title:string author:string content:text posted_at:datetime
% rake db:migrate
% rake
% rake
LOADED SUITE test,test/functional,test/performance,test/unit/helpers,test/unit

ActionController::TestCase
ActionDispatch::IntegrationTest
ActionView::TestCase
ActiveRecord::TestCase
ActiveSupport::TestCase
PostTest
PostsHelperTest
Test::Unit::TestCase
==============================================================================
  pass: 0,  fail: 0,  error: 0
  total: 0 tests with 0 assertions in 0.003729 seconds
==============================================================================
LOADED SUITE test,test/functional,test/performance,test/unit/helpers,test/unit

ActionController::TestCase
ActionDispatch::IntegrationTest
ActiveRecord::TestCase
ActiveSupport::TestCase
PostsControllerTest
     test_should_create_post                                              PASS
     test_should_destroy_post                                             PASS
     test_should_get_edit                                                 PASS
     test_should_get_index                                                PASS
     test_should_get_new                                                  PASS
     test_should_show_post                                                PASS
     test_should_update_post                                              PASS
Test::Unit::TestCase
==============================================================================
  pass: 7,  fail: 0,  error: 0
  total: 7 tests with 10 assertions in 0.1396 seconds
==============================================================================
```

```
% cd rails_3.0.12
% bundle install
% rails g scaffold post title:string author:string content:text posted_at:datetime
% rake db:migrate
% rake
Run options:

# Running tests:

.

Finished tests in 0.033918s, 29.4829 tests/s, 29.4829 assertions/s.

1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
Run options:

# Running tests:

.......

Finished tests in 0.194761s, 35.9415 tests/s, 51.3450 assertions/s.

7 tests, 10 assertions, 0 failures, 0 errors, 0 skips
```

All the tests are all working fine. No gemsets required! Now let's try actually
running `rails console`.

```
% cd rails_3.2.2
% rails c
Loading development environment (Rails 3.2.2)
1.9.3p0 :001 > ^D

% cd ../rails_3.1.4
% rails c
Loading development environment (Rails 3.1.4)
1.9.3p0 :001 > ^D

% cd ../rails_3.0.12
% rails c
Loading development environment (Rails 3.0.12)
1.9.3p0 :001 > ^D
```

Well, that was easy. Rails uses bundler very well, and we are isolated from the
actual Ruby environment by the Rails console and other scripts. Now let's see
how a non-Rails Ruby project behaves.

## Bundling a typical Ruby project

We'll use Savon as our testing gem for non-Rails projects. It's simple but has
some dependencies, it has a solid version history to go back into, and it's
where I look whenever I want to see some good Ruby gem code.

### Savon 0.9.9

First we'll setup a project with the latest version of Savon.

```
% mkdir savon_0.9.9
% cd savon_0.9.9
% bundle init
```

Change the Gemfile:

```ruby
source "https://rubygems.org"
gem 'savon', '~> 0.9.9'
```

### Savon 0.8.6

Now we'll setup a project with the previous major version of Savon.

```
% mkdir savon_0.8.6
% cd savon_0.8.6
% bundle init
```

Change the Gemfile:

```ruby
source "https://rubygems.org"
gem 'savon', '~> 0.8.6'
```

There, now we have one project with a Gemfile using Savon 0.8.6 and one
using Savon 0.9.9. But how easy is it to develop in an environment with the
gem versions all mixed together?

```
% cd savon_0.9.9
% irb
1.9.3p0 :001 > require 'savon'
 => true
1.9.3p0 :002 > Savon::Version
 => "0.9.9"
```

```
% cd savon_0.8.6
% irb
1.9.3p0 :001 > require 'savon'
 => true
1.9.3p0 :002 > Savon::Version
 => "0.9.9"
```

Whoops. `irb` doesn't know or care about our Gemfile specifying the gem versions
we want to use. That means that a Ruby script won't care either.

```ruby
require 'savon'
puts Savon::Version
```

Copy that script into each project directory and run it with `ruby version.rb`.
For each directory the output will be the same: `0.9.9`.

So what can we do?

### Use Bundler commands to have Ruby follow the Gemfile

`bundle exec` will run the given command within the context of the local Gemfile. That means
that `bundle exec ruby` will run Ruby with the correct gem versions.

```
# in savon_0.9.9
% bundle exec ruby version.rb
0.9.9

# in savon_0.8.6
% bundle exec ruby version.rb
0.8.6
```

`bundle console` starts up an interactive Ruby session within the context of the
local Gemfile so it's exactly what we need to replace `irb`. It's nifty in that
it also automatically includes any gems specified in the Gemfile, no need to
`require`!

```
# in savon_0.9.9
% bundle console
1.9.3p0 :001 > Savon::Version
 => "0.9.9"

# in savon_0.8.6
% bundle console
1.9.3p0 :001 > Savon::Version
 => "0.8.6"
```

### Aside: the bundler plugin in ZSH handles bundler exec automatically

If you're using ZSH then you can add the bundler plugin and avoid some of these
hoops. It has a function that checks to see if bundler is installed and if
you're in a directory with a Gemfile. If so, then if you run a command on its
list of commands to run with bundle exec (e.g. cap, guard, heroku, ruby, rake)
it will automatically wrap that command in bundle exec. Win. With that plugin
in place, you can almost entirely just stop using gemsets and you'll never have
any problems.

## Conclusion

André Arko was totally right. If you're already using bundler to manage the gems
for all of your projects (and you should) then you can just stop dealing with
gemsets and let bundler sort it all out for you.

It seems to me that the time of gemsets has come and gone. With bundler as fast
and easy to use as it is, there's really no need to maintain gemsets for each
project.

I for one -- with over 30 gemsets -- am pleased that it all just works
so well and easily. I can get a specific gem version's binary pretty easily
(and that's a rare need) and my ZSH setup abstracts away the headaches.

Not only will this eliminate the work involved in maintaining gemsets, bundler
will be able to pull from the common collection of local gems when satisfying
dependencies. No more pulling down new copies of all the required gems whenever
I start a new project.
