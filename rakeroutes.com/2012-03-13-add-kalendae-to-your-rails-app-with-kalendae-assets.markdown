---
layout: single
title: "Add Kalendae to your Rails app with Kalendae Assets"
date: 2012-03-13 19:24
comments: true
categories: rails
---

![kalendae picker](/assets/images/kalendae_post/kalendae_picker.png){: .align-right}

[Kalendae](https://github.com/ChiperSoft/Kalendae) is an awesome JavaScript date
picker that doesn't require jQuery (or any other library) and that just works. I
liked it so much that I wrote a gem to make it easy to add to the Rails asset
pipeline: [Kalendae Assets](https://github.com/sdball/kalendae_assets)

In this post, I show just how easy it is to add Kalendae to an existing Rails
app and end up with a great looking date picker.

<!-- more -->

## Set up a Rails Application for Kalendae

First off, you'll need a 3.1+ Rails app.

```
rvm use --create 1.9.3@kalendae_assets_demo
gem install rails
rails new kalendae_assets_demo
cd kalendae_assets_demo
```

Next, scaffold a model that has a date field.

```
rails g scaffold videogame title release_date:date
rake db:migrate
```

If you were to run your Rails application now and create a new videogame you'd
see the standard three input date picker.

![kalendae scaffold](/assets/images/kalendae_post/scaffold.png){: .align-right}

That's ok to get started, but now it's time kick it up a notch. Bam!

## Add Kalendae to your Rails App

To make Kalendae available to your app, you just need to install the
`kalendae_assets` gem into the `assets` bundle group.

```ruby
source 'https://rubygems.org'

gem 'rails', '3.2.2'

gem 'sqlite3'

group :assets do
  gem 'sass-rails',   '~> 3.2.3'
  gem 'coffee-rails', '~> 3.2.1'
  gem 'uglifier', '>= 1.0.3'
  gem 'kalendae_assets' # <--------- right here
end

gem 'jquery-rails'
```

Once you've got the file edited it's time for a bundle update.

```
bundle
# (snip lots of "Using")
Installing kalendae_assets (0.1.1)
```

Now add `//= require kalendae` to your `application.js` file.

```
//= require jquery
//= require jquery_ujs
//= require kalendae
//= require_tree .
```

And `*= require kalendae` to your `application.css` file.

```
*= require_self
*= require_tree .
*= require kalendae
```

## Add a Kalendae date picker using CoffeeScript (or JavaScript)

To add a Kalendae date picker to the input on the Videogame form we'll have to
attach it with some CoffeeScript so we can customize the date format.
Fortunately, if jQuery is available then Kalendae provides a jQuery plugin to
make things just about as easy as possible.

Give your element some class or id to hook onto, e.g. `release_date`

```erb
<div class="field">
  <%= f.label :release_date %><br />
  <%= f.text_field :release_date, class: 'release_date' %>
</div>
```

Next edit your app to hook Kalendae into inputs with the `release_date` class.
For this example we'll just use the `videogames.js.coffee` file.

```coffeescript
$(document).ready ->
  $('input.release_date').kalendae({
    format: 'YYYY-MM-DD'
  })
```

Besides the date format, Kalendae has a bunch of options you can pass to the
picker. Check out
[the Kalendae documentation](https://github.com/ChiperSoft/Kalendae)) for full details.

And, that's it. Now our scaffolded form looks a bit snazzier.

![kalendae input](/assets/images/kalendae_post/kalendae.png)
