---
layout: single
title: "Deploying Octopress"
date: 2012-02-05 11:05
comments: true
categories: software
---

The easiest way to deploy Octopress out of the box is, by far, rsync. Why do I
think it's easier than github pages or heroku?

<!-- more -->

Because I already have a top-level github page setup with a custom domain. I have
sdball.github.com setup as http://stephenballnc.com. Having a project page there
works great. If I have a project "froboz" and create a gh-pages branch then github will easily
serve it up as http://stephenballnc.com/froboz. Awesome! Having a project page get its own
custom domain should work fine but seems really weird.

## Github Pages

I tried for some time to get Octopress deployed to a private repo's github
project page with a custom domain. But I could never quite wrap my head around
the codebase containing the code to generate the blog and the generated blog
itself.

## Heroku

Similarly Heroku. If you want to mix Octopress with Heroku, Guillermo Esteves
nicely explains how he handled his [heroku deploy of Octopress](http://blog.gesteves.com/2011/09/19/hello-world/).
But still you end up with a repo containing the blog source code and the
generated html. Yes you can do some gymnastics in .slugignore to keep the heroku
app from bulking up it still seems weird. Also spinning up a heroku instance
just to serve static files just doesn't seem right.

Although, if you think you'll ever want to expand beyond static blog content.
Then getting the blog onto Heroku makes sense. On Heroku you'd be free to add
whatever Rack application you'd care to write.

## Jekyll Bootstrap

For a time I even explored using [Jekyll Bootstrap](http://jekyllbootstrap.com/) since one of its core philosophies is
that the repo shouldn't store the generated code and the blogging compilation
should ideally be handled by Github. I brought it up, played around, contributed
a tiny piece of code upstream, but I missed a lot of the Octopress niceties.

And by niceties I mean, plugins. Jekyll Bootstrap is a cool idea and all, but
it's designed to be a pure Jekyll implementation with no plugins so that github
can handle the site compilation for us. But the Octopress plugins are great!

Jekyll-Bootstrap looks very promising, but I don't really mind generating the
html of my blog on my machine; I just don't want to store it in my repo.

## rsync

So rsync. It's as easy as the Octopress docs would have you believe. As long as
you already have hosting somewhere. I'm on Dreamhost and have been very happy
with their hosting. Except Rails hosting, that's Heroku.

1. Set the configuration variables for your ssh server
2. `rake gen_deploy`

Done. And done *FAST*! Using rsync is still geeky enough for me and I can rest easy
knowing that this blog is just being served as static files. Handy should this
blog explode in popularity.
