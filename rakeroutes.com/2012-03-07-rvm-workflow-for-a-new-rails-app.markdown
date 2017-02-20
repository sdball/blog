---
layout: single
title: "RVM workflow for a new Rails app"
date: 2012-03-07 21:37
comments: true
categories: [ruby, snippets]
---

You want to start a new Rails app. But you also want to start a new RVM gemset
for the app so you can start with the latest Rails and gems. In this code
snippet I show how I start off an up-to-date Rails project in a clean gemset.

<!-- more -->

## Creating a new Rails app

```
# create and use the new RVM gemset
$ rvm use --create 1.9.3@awesome_rails_project

# install rails into the blank gemset
$ gem install rails

# generate the new rails project
$ rails new awesome_rails_project

# go into the new project directory and create an .rvmrc for the gemset
$ cd awesome_rails_project
$ rvm --rvmrc 1.9.3@awesome_rails_project

# verify the rvmrc
$ cd ..; cd -
```

There you go. A blank slate gemset and the latest Rails, just waiting for you to
code in some awesome.
