---
layout: single
title: "Let's Write a Gem: Part 2"
date: 2012-02-23 12:10
comments: true
categories: ruby
---

Continued from [Let's Write a Gem, Part 1](http://rakeroutes.com/blog/lets-write-a-gem-part-one/)

In this post we'll finish the gem. Along the way I'll point out some confusing
pitfalls you can run into when testing gems locally.

<!-- more -->

## 9. Behavior Driven Development

Next up it's time to pick a testing framework for our Gem. Let's go with RSpec
just because.

### 9.1 Modify the gemspec to include rspec and rake.

> File: periodic_table.gemspec

```ruby
# -*- encoding: utf-8 -*-
$:.push File.expand_path("../lib", __FILE__)
require "periodic_table/version"

Gem::Specification.new do |s|
  s.name        = "periodic_table"
  s.version     = PeriodicTable::VERSION
  s.authors     = ["Stephen Ball"]
  s.email       = ["sdball@gmail.com"]
  s.homepage    = ""
  s.summary     = %q{Provide periodic table data.}
  s.description = %q{Provide data on elements in the periodic table.}

  s.rubyforge_project = "periodic_table"

  s.files         = `git ls-files`.split("\n")
  s.test_files    = `git ls-files -- {test,spec,features}/*`.split("\n")
  s.executables   = `git ls-files -- bin/*`.split("\n").map{ |f| File.basename(f) }
  s.require_paths = ["lib"]

  s.add_development_dependency 'rake'
  s.add_development_dependency 'rspec'

  # s.add_runtime_dependency "rest-client"
end
```

### 9.2 Modify the Rakefile to include rspec and default to test

This will let us just run `rake` to test our gem.

> File: Rakefile

```ruby
require "bundler/gem_tasks"
require "rspec/core/rake_task"

RSpec::Core::RakeTask.new

task :default => :spec
task :test => :spec
```

### 9.3 Run bundle update to install the gems for development

```
% bundle update
Fetching source index for http://rubygems.org/
Installing rake (0.9.2.2)
Installing diff-lcs (1.1.3)
Using periodic_table (0.0.1) from source at /Users/stephenball/github/sdball/periodic_table
Installing rspec-core (2.8.0)
Installing rspec-expectations (2.8.0)
Installing rspec-mocks (2.8.0)
Installing rspec (2.8.0)
Using bundler (1.0.22)
```

There. Now running `rake` will run our specs.

```
% rake
No examples matching ./spec{,/*/**}/*_spec.rb could be found
```

## 10. Write some tests

To keep this tutorial about the gem, let's keep the testing very simple.

- Create a `/spec` directory
- Create `/spec/spec_helper.rb`

```ruby
require 'periodic_table'
```

- Create `/spec/lib/periodic_table_spec.rb`

```ruby
require 'spec_helper'

describe PeriodicTable do
  it "should return data for a named element" do
    element_data = PeriodicTable.lookup('oxygen')
    element_data.should_not be_nil
    element_data.symbol.should == 'O'
    element_data.atomic_weight.should == '15.9994'
  end
end
```

Now running rake shows that our tests are being loaded, executed, and our gem is
being automatically pulled in.

```
% rake
/Users/stephenball/.rvm/rubies/ruby-1.9.3-p0/bin/ruby -S rspec ./spec/lib/periodic_table_spec.rb
F

Failures:

  1) PeriodicTable should return data for a named element
     Failure/Error: element_data = PeriodicTable.lookup('oxygen')
     NoMethodError:
       undefined method `lookup' for PeriodicTable:Module
     # ./spec/lib/periodic_table_spec.rb:5:in `block (2 levels) in <top (required)>'

Finished in 0.00037 seconds
1 example, 1 failure

Failed examples:

rspec ./spec/lib/periodic_table_spec.rb:4 # PeriodicTable should return data for a named element
rake aborted!
/Users/stephenball/.rvm/rubies/ruby-1.9.3-p0/bin/ruby -S rspec ./spec/lib/periodic_table_spec.rb failed

Tasks: TOP => default => spec
(See full trace by running task with --trace)
```

Of course running `rspec spec` will work too.

**Note**: you don't need to have the gem installed in order to run the
tests. The files are loaded and tested directly from /lib. This is great because
it really encourages a good "red, green, refactor" development process.

If you really want to run the gem in the irb and you don't want to deal with
uninstall/reinstall then you can just run `bundle console`. That command will
land you in a console environment with the gem already loaded.

## 11. Write the gem to pass the tests.

Hey, this is the actual gem writing part!

Ok, we'll use savon to talk to the periodic table SOAP API, so that dependency
is next.

- add savon as a runtime dependency

> File: periodic_table.gemspec

```ruby
# -*- encoding: utf-8 -*-
$:.push File.expand_path("../lib", __FILE__)
require "periodic_table/version"

Gem::Specification.new do |s|
  s.name        = "periodic_table"
  s.version     = PeriodicTable::VERSION
  s.authors     = ["Stephen Ball"]
  s.email       = ["sdball@gmail.com"]
  s.homepage    = ""
  s.summary     = %q{Provide periodic table data.}
  s.description = %q{Provide data on elements in the periodic table.}

  s.rubyforge_project = "periodic_table"

  s.files         = `git ls-files`.split("\n")
  s.test_files    = `git ls-files -- {test,spec,features}/*`.split("\n")
  s.executables   = `git ls-files -- bin/*`.split("\n").map{ |f| File.basename(f) }
  s.require_paths = ["lib"]

  s.add_development_dependency 'rake'
  s.add_development_dependency 'rspec'

  s.add_runtime_dependency 'savon'
end
```

- bundle install to get savon

```
bundle install
```

Now we just need to code up enough of the API to pass the test.


- create `lib/periodic_table/periodic_table_api.rb`

> periodic_table_api.rb

```ruby
require 'savon'

module PeriodicTable
  class PeriodicTableApi
    def initialize
      @client = Savon::Client.new do
        wsdl.document = 'http://www.webservicex.net/periodictable.asmx?WSDL'
      end
    end

    def query(element_name)
      api_response = @client.request :get_atomic_number do
        soap.body = {'ElementName' => element_name}
      end
      result = api_response.to_hash[:get_atomic_number_response][:get_atomic_number_result]
      ApiResponse.new(result)
    end
  end

  # Wow, this is ugly. I did not expect nested XML.
  class ApiResponse
    attr_reader :atomic_weight,
                :symbol,
                :atomic_number,
                :element_name,
                :boiling_point,
                :ionisation_potential,
                :electro_negativity,
                :atomic_radius,
                :melting_point,
                :density

    def initialize(result)
      xml = Nokogiri::XML.parse(result)
      @atomic_weight = xml.at('AtomicWeight').text
      @symbol = xml.at('Symbol').text
      @atomic_number = xml.at('AtomicNumber').text
      @element_name = xml.at('ElementName').text
      @boiling_point = xml.at('BoilingPoint').text
      @ionisation_potential = xml.at('IonisationPotential').text
      @electro_negativity = xml.at('EletroNegativity').text
      @atomic_radius = xml.at('AtomicRadius').text
      @melting_point = xml.at('MeltingPoint').text
      @density = xml.at('Density').text
    end
  end
end
```

- add the periodic_table_api to the main gem file

> periodic_table.rb

```ruby
require 'periodic_table/version'
require 'periodic_table/periodic_table_api'

module PeriodicTable
  def self.lookup(element_name)
    PeriodicTableApi.new.query(element_name)
  end
end
```

```
% rake

# [snip lots of savon output]

Finished in 1.27 seconds
1 example, 0 failures
```
Yeah! Passing tests (ok test). Must be time to launch.

### Aside: bundle console

While developing a gem don't forget about that `bundle console` command. Running
it will drop you in an irb session with the current **bundler** (not system)
environment loaded. Since you're in a gem with a good gemspec, that means that
the PeriodicTable files will all be already loaded. Check it out.

```
% irb
1.9.3p0 :001 > PeriodicTable
NameError: uninitialized constant PeriodicTable
  from (irb):1
  from /Users/stephenball/.rvm/rubies/ruby-1.9.3-p0/bin/irb:16:in `<main>'
1.9.3p0 :001 > require 'periodic_table'
LoadError: cannot load such file -- periodic_table
  from /Users/stephenball/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in `require'
  from /Users/stephenball/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in `require'
  from (irb):1
  from /Users/stephenball/.rvm/rubies/ruby-1.9.3-p0/bin/irb:16:in `<main>'
```

```
% bundle console
1.9.3p0 :001 > PeriodicTable
 => PeriodicTable
```

This is super handy while you are writing your gem and you want to dive into a
piece of it without having to jump through any hoops to get the lib directory
loaded.

## 12. Test the gem locally

Ok. We've got a gem that's ready for release. This is where all most gem writing
tutorials will jump right to putting gems on RubyGems.org. Now RubyGems is amazing
and awesome, but you might not be ready for the public to get to this code. What
if you want to test your gem locally? What if you want to keep your gem in a private github repo? No problem! Here's how to keep on rolling.

Let's say we're in another project on the same machine as the gem we just
developed. The magic of bundler means we can just include the new gem's path
right in that project's Gemfile. Similarly you can push the gem git repo up to a
private git repository and put that repo URL in the Gemfile.

- `:path` - load the gem from a local path
- `:git` - load the gem from an accessible git repo

A neat thing about adding your gems this way is that you won't need to keep
rebuilding the gem with the `rake build` or `rake install` commands because the
gem files will be loaded *directly*. Just like they are when we run our rake
tests.

### 12.1 Include the gem in a Gemfile with :path or :git

> Gemfile

```ruby
gem 'periodic_table', :path => '~/github/sdball/periodic_table'
```

> Gemfile

```ruby
# this will actually work if you want to try it on your machine
gem 'periodic_table', :git => 'git://github.com/sdball/periodic_table.git'
```

After that, just a quick `bundle install` will ensure that the gem source is all
loaded and ready to roll.

There's just one thing to keep in mind. Because the gem won't actually be
installed to the "system" gems (i.e. your rvm gemset) it won't be listed in `gem
list` and won't be directly loadable in an irb session or a script.

Let me repeat that, because I spent way too much time going crazy trying to
figure this out.

When you install a gem via `:path` or `:git` it will not be listed in `gem list`
and will throw a LoadError if you try to require it directly in irb.

What's the answer? How can you actually use the gem? Two bundler commands.

### 12.2 Start irb and run scripts using the bundle command

- `bundle exec`: if you want to run a script that requires your gem
- `bundle console`: if you want to use your gem in irb

"But Stephen", you might say, "you've already told me about bundle console twice
already."

Yes, because it's really frustrating when you have a gem that has passing tests
and runs beautifully from a `rake install`, but will simply not behave when you
try to run it after dropping a `:path` or `:git` to it from other project's
Gemfile.

#### bundle exec

Say you've got the following contrived script.

> oxygen_weight.rb

```ruby
require 'periodic_table'

puts PeriodicTable.lookup('oxygen').atomic_weight
```

Nothing fancy. But if we've included the periodic_table gem via `:git` or `:path` it
will absolutely not work if we try and run it directly.

```
% ruby oxygen_weight.rb
/Users/stephenball/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in `require': cannot load such file -- periodic_table (LoadError)
	from /Users/stephenball/.rvm/rubies/ruby-1.9.3-p0/lib/ruby/site_ruby/1.9.1/rubygems/custom_require.rb:36:in `require'
	from oxygen_weight.rb:1:in `<main>'
```

Bundle exec to the rescue!

```
% bundle exec ruby oxygen_weight.rb

# [snip savon output (I really should hide that)]

15.9994
```

Woo!

#### bundle console

Just like we've already gone over. `bundle console` will load up an irb session
in your current bundler environment. That is, all the gems specified in the
Gemfile will be loaded and not just the system gems.

```
% bundle console
1.9.3p0 :001 > PeriodicTable
 => PeriodicTable
1.9.3p0 :001 > PeriodicTable.lookup('oxygen').atomic_radius
 # [snip savon output]
 => "0.74"
```

## 13. Release to RubyGems.org

Ok, I guess this tutorial wouldn't be complete without the requisite releasing
of the gem to RubyGems.

```
# in the gem development directory
% rake release
```

Bundler will walk you through everything you need from there.

It really is that easy.

- [PeriodicTable on RubyGems](https://rubygems.org/gems/periodic_table)
- [PeriodicTable on Github](https://github.com/sdball/periodic_table)

## Wrap it up

Well readers, I hope this long two part tutorial hasn't completely scared you
away from the Ruby gem creation process. It really is very easy, there are just
a few steps and practices to keep in mind.

Once you have the process down though, you can actually take a gem from idea to
worldwide distribution as fast as you can code it. That's absolutely
astonishing and one of my favorite aspects of Ruby. Anyone, *anyone* can write
some code and have it hosted on a high-speed, always-on, super easy to use code
distribution platform available to every single Ruby developer out there with
an Internet connection. Think about that. What an awesomely powerful aspect of
our community!

So get out there and share an idea. :-)

Check out the ["Let's Write a Gem" Reddit discussion](http://www.reddit.com/r/ruby/comments/q2t4m/rruby_writing_gems_is_easy_and_fun_do_it/)
