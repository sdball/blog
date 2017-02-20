---
title: "Let's Use Hwacha to Scan URLs"
date: 2013-12-18 12:00
---

I've written a gem, [Hwacha](https://github.com/sdball/hwacha), as a wrapper around Typhoeus to allow for quick and easy response checking for multiple URLs. Let's go through some simple use cases!

I started to write a script to go through Ruby Weekly issues and add them to my pinboard. I wanted to do this so I could use pinboard's awesome fulltext archive to easily find interesting articles weeks later. Fun!

I've also really taken Donald's [Dependency Isolation](https://speakerdeck.com/triangleruby/dependency-isolation-by-donald-ball) talk to heart. So when I started to build out my simple script I began by isolating the dependencies like the library, Typhoeus, that I'd use to hit the URLs and check their response.

Well one thing led to another and I ended up just turning that first dependency isolation into its own gem: [Hwacha](https://github.com/sdball/hwacha)! At it's core Hwacha is just a wrapper around Typhoeus's `hydra` method and is just setup to yield the response and URL to the given block. I also added `find_existing` that only executes the block if the page exists.

So that's cool, but what are some things we can do with it? How about that importing Ruby Weekly issues project?

## Import Ruby Weekly Issues into Pinboard

`Bookmarks` and `RubyWeekly` are my dependency isolations for the Pinboard API and for generating the array of potential Ruby Weekly issue URLs. They aren't perfect, but they got the job done (and were tested)!

```ruby
# bookmarks.rb
require 'pinboard'
require 'delegate'

module PinboardCredentials
  USERNAME = 'xyzzyb'
  PASSWORD = '[redacted]'

  def self.hash
    {
      :username => USERNAME,
      :password => PASSWORD,
    }
  end
end

class Bookmarks < SimpleDelegator
  include PinboardCredentials

  def initialize
    super(Pinboard::Client.new(PinboardCredentials.hash))
  end
end
```

```ruby
# ruby_weekly.rb
require 'open-uri'

class RubyWeekly
  def self.base_url
    'http://rubyweekly.com'
  end

  def self.issue_url(number)
    "#{base_url}/issues/#{number}"
  end

  def self.potential_issue_urls
    1.upto(current_issue_number).map do |number|
      issue_url(number)
    end
  end

  def self.current_issue_number
    Integer(landing_page_text.match(/issues\/(\d+)/).to_a.last)
  end

  def self.landing_page_text
    open('http://rubyweekly.com').read
  end
end
```

```ruby
require 'hwacha'
require_relative '../lib/bookmarks'
require_relative '../lib/rubyweekly'

bookmarks = Bookmarks.new
hwacha = Hwacha.new

hwacha.check(RubyWeekly.potential_issue_urls) do |url, response|
  next if response.body == 'no such issue'

  number = url.scan(/\d+/).first
  description = "Ruby Weekly #{number}"
  tags = 'petercooper,ruby,rubyweekly'

  bookmarks.add(:url => url, :description => description, :tags => tags)

  p "Added #{url} ^_^"
end
```

I'd have liked to use the `find_existing` method, but rubyweekly.com responds with a successful http response and the body "no such issue" for any issue url that doesn't exist. But by design, Hwacha lets us dig into the response so we can make smart decisions.

## Check responses for all URLs on a given page

Here's a fun one, let's scan all the URLs we can find on a given webpage.

```ruby
require 'open-uri'

page_text = open('http://rakeroutes.com').read
Hwacha.new.check(page_text.scan(/http:\/\/[^" ;]+\b/)) do |url, response|
  puts "#{url} is invalid!" unless response.success? || response.code == 302
end
```

Or maybe we want to see all the nice successful URLs.

```ruby
require 'open-uri'

page_text = open('http://rakeroutes.com').read
Hwacha.new.find_existing(page_text.scan(/http:\/\/[^" ;]+\b/)) do |url|
  puts "#{url} is valid!"
end
```

Since it's just a wrapper around Typhoeus it means that Hwacha is actually got a first class response object to work with. We could even dig more into the response body for some content checking or light page scraping.

```ruby
require 'open-uri'

page_text = open('http://rakeroutes.com').read
Hwacha.new.find_existing(page_text.scan(/http:\/\/[^" ;]+\b/)) do |url|
  puts response.body[0..50]
end
```

