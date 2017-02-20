---
layout: single
title: "How to write (and test) a gem to serve static files on the Rails asset pipeline"
date: 2012-03-15 10:46
comments: true
categories: [ruby, rails]
---

Today we write (and test!) a gem that simply adds new static assets to a Rails
project. I puzzled out most of this while working on my kalendae_assets gem,
then discovered that most of the work gets done for you when you just run
`rails plugin new`.

<!-- more -->

Why write a gem to serve static assets?

- It will allow you to have less crap in your Rails app. Sure you can throw all
  the files in `/vendor`, but this is even better! You know how you don't have
  to add jquery.min.js to your Rails app anymore? Yeah.
- You can easily guarantee that your projects are using a consistent version of
  whatever resources you care about.
- It's pretty awesomely easy to do once you get the steps down.

So let's get going.

## Step 1: Get Going

```
% rvm use --create 1.9.3@static_assets
% gem install rails
% rails plugin new static_assets
% cd static_assets
% rvm --rvmrc 1.9.3@static_assets
% cd ..; cd -
# accept the .rvmrc
% gem install bundler
% bundle install
# isn't that new faster bundler great?
```

Now, take a look at what we've got here.

```
. # /static_assets
├── Gemfile
├── Gemfile.lock
├── MIT-LICENSE
├── README.rdoc
├── Rakefile
├── lib
│   ├── static_assets
│   │   └── version.rb
│   ├── static_assets.rb
│   └── tasks
│       └── static_assets_tasks.rake
├── static_assets.gemspec
└── test
    ├── dummy
    │   ├── README.rdoc
    │   ├── Rakefile
    │   ├── app
    │   │   ├── assets
    │   │   │   ├── images
    │   │   │   ├── javascripts
    │   │   │   │   └── application.js
    │   │   │   └── stylesheets
    │   │   │       └── application.css
    │   │   ├── controllers
    │   │   │   └── application_controller.rb
    │   │   ├── helpers
    │   │   │   └── application_helper.rb
    │   │   ├── mailers
    │   │   ├── models
    │   │   └── views
    │   │       └── layouts
    │   │           └── application.html.erb
    │   ├── config
    │   │   ├── application.rb
    │   │   ├── boot.rb
    │   │   ├── database.yml
    │   │   ├── environment.rb
    │   │   ├── environments
    │   │   │   ├── development.rb
    │   │   │   ├── production.rb
    │   │   │   └── test.rb
    │   │   ├── initializers
    │   │   │   ├── backtrace_silencers.rb
    │   │   │   ├── inflections.rb
    │   │   │   ├── mime_types.rb
    │   │   │   ├── secret_token.rb
    │   │   │   ├── session_store.rb
    │   │   │   └── wrap_parameters.rb
    │   │   ├── locales
    │   │   │   └── en.yml
    │   │   └── routes.rb
    │   ├── config.ru
    │   ├── db
    │   ├── lib
    │   │   └── assets
    │   ├── log
    │   ├── public
    │   │   ├── 404.html
    │   │   ├── 422.html
    │   │   ├── 500.html
    │   │   └── favicon.ico
    │   ├── script
    │   │   └── rails
    │   └── tmp
    │       └── cache
    │           └── assets
    ├── static_assets_test.rb
    └── test_helper.rb
```

Yep, it's a gem! And is that a familiar looking directory layout in
`test/dummy`? What in the world is that, you might ask. That's actually a
complete Rails application for testing your Rails gem. Awesome.

That included Rails application is pretty slick and it was my biggest stumbling
block when I was casting about trying to figure out how to actually *test*
my Kalendae Assets gem. The solution to include an actual Rails app for testing
might seem crazy at first, but it makes sense. How better to check that your
Rails gem works in Rails?

## Step 2: Switch over to minitest and Capybara

*I'm going to skip or skim right over all the standard gem writing pieces. If you need
a refresher there check out
[Let's Write a Gem, Part 1](http://rakeroutes.com/blog/lets-write-a-gem-part-one/).*

Instead of RSpec, this time let's switch this Gem over to minitest for our
testing framework. We'll also use Capybara for our integration testing.

### 2.1: Add the minitest and Capybara gems

```ruby
$:.push File.expand_path("../lib", __FILE__)

# Maintain your gem's version:
require "static_assets/version"

# Describe your gem and declare its dependencies:
Gem::Specification.new do |s|
  s.name        = "static_assets"
  s.version     = StaticAssets::VERSION
  s.authors     = ["TODO: Your name"]
  s.email       = ["TODO: Your email"]
  s.homepage    = "TODO"
  s.summary     = "TODO: Summary of StaticAssets."
  s.description = "TODO: Description of StaticAssets."

  s.files = Dir["{app,config,db,lib}/**/*"] + ["MIT-LICENSE", "Rakefile", "README.rdoc"]
  s.test_files = Dir["test/**/*"]

  s.add_dependency "rails", "~> 3.2.2"

  s.add_development_dependency "sqlite3"
  s.add_development_dependency 'minitest' # <------- here
  s.add_development_dependency 'capybara' # <------- and here
end
```

Then rerun `bundle install` to get the gems pulled in.

### 2.2: Switch the test helper to minitest/Capybara

Change the `/test/test_helper.rb` file to the following.

```ruby
# Configure Rails Environment
ENV['RAILS_ENV'] = 'test'
require File.expand_path('../dummy/config/environment.rb',  __FILE__)
require 'rails/test_help'
require 'minitest/autorun'
require "capybara/rails"

Rails.backtrace_cleaner.remove_silencers!

class IntegrationTest < MiniTest::Spec
  include Capybara::DSL
  register_spec_type(/integration$/, self)
end
```

That `register_spec_type` means that any `describe` blocks ending in
"integration" will be matched as an IntegrationTest and get the Capybara DSL.

Checkout the first few lines of that file. That's some of cool magic brought in
by `rails plugin new`. Set the Rails environment to test and load the app, it
seems so simple in retrospect.

### 2.3: Remove the static_assets_test.rb file

We'll setup our tests a bit differently than the plugin assumed. Remove the file
`static_assets_test.rb` and create a directory under `test` called
`integration`.

## Step 3: Write some tests!

In the new `test/integration` directory, create a file called `static_assets_integration_test.rb`

```ruby
require 'test_helper'

describe "static assets integration" do
  it "provides our_awesome_static_asset.js on the asset pipeline" do
    visit '/assets/our_awesome_static_asset.js'
    page.text.must_include 'var StaticAsset = {};'
  end

  it "provides our_awesome_static_asset.css on the asset pipeline" do
    visit '/assets/our_awesome_static_asset.css'
    page.text.must_include '.static_asset {'
  end
end
```

These tests should be enough to verify that the files are being served
correctly.

### 3.1: Validate the tests by checking that they fail

```
% rake
1) Error: test_0001_provides_our_awesome_static_asset_js_on_the_asset_pipeline(static assets integration):
ActionController::RoutingError: No route matches [GET] "/assets/our_awesome_static_asset.js"

2) Error: test_0002_provides_our_awesome_static_asset_css_on_the_asset_pipeline(static assets integration):
ActionController::RoutingError: No route matches [GET] "/assets/our_awesome_static_asset.css"
```

Yeah! Let's make some tests green.

## Step 4: Add the static files

### 4.1 Create the directory structure

Create a new directory under `static_assets` called `vendor`. In this new
directory create a directory `assets`. Under `assets` create three directories:
`images`, `javascripts`, and `stylesheets`.

```
# this command will do it for you if you're in the static_assets directory
% mkdir -p vendor/assets/{images,javascripts,stylesheets}
```

It should all look like this.

```
. # static_assets
└── vendor
    └── assets
        ├── images
        ├── javascripts
        └── stylesheets
```

In our case, we won't actually use the images directory but I'm showing it here
for reference.

### 4.2 Add the files

Add this file to `javascripts`

```js
var StaticAsset = {};
```

Add this file to `stylesheets`

```css
.static_asset {
  padding: 20px;
}
```

Hey, just enough to pass the tests. (Once we have the files in the asset
pipeline.)

## Step 5: Make a Rails engine to serve up the static files on the asset pipeline

Remember that gem structure in `lib`? Now we actually do something in there.

### 5.1: Create engine.rb

Create a new file, `lib/static_assets/engine.rb`

```ruby
module StaticAssets
  class Engine < ::Rails::Engine
    initializer 'static_assets.load_static_assets' do |app|
      app.middleware.use ::ActionDispatch::Static, "#{root}/vendor"
    end
  end
end
```

There's a lot of magic packed into these lines. Essentially we've just made a
little stub of middleware to statically serve up the files in `/vendor`. When a
Rails app asks the asset pipeline for the files for pulling into
`application.js` or `application.css` our gem will be there.

### 5.2: Load engine.rb

Change `lib/static_assets.rb` to require `engine.rb`

```ruby
require "static_assets/engine"

module StaticAssets
end
```

And with that, we're done!

## Step 6: Run the tests

At this point, our tests should be green.

```
% rake
Run options:

# Running tests:

..

Finished tests in 0.408642s, 4.8943 tests/s, 9.7885 assertions/s.

2 tests, 4 assertions, 0 failures, 0 errors, 0 skips
```

Now (if we were to release this as a gem) any Rails project would just have to
add this to their Gemfile:

```ruby
group :assets do
  gem 'static_assets'
end
```

And add `//= require our_awesome_static_asset` to their application.js and
`*= require our_awesome_static_asset` to their application.css. Pretty easy!

If you want to reference an actual gem that I've written with this technique,
check out [Kalendae Assets](https://github.com/sdball/kalendae_assets)
