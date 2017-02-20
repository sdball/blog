---
title: "Getting Up to Speed on a New Git Repo"
date: 2012-05-11 12:00
excerpt: "Want to get up to speed with a new git repository? Here are some tips and tools to quickly see what's been going on."
---

Alright! You want to get up to speed with a new git repository? You got it. Here are some quick reference notes and tools to use to see what's been going on.

**Caveat**: I'm a little rushed writing this so it's going to be more of a highlights post than a fully detailed tutorial. If I feel like there's a lot of material that I'm having to skip I'll make more detailed posts on specific points. Or maybe I'll write up a PDF or epub for handy reference.

Gary Bernhardt is pretty much the guy here. If you want to really get to know git and learn some amazing tricks for working with a git repo as efficiently as you work with code then you should subscribe to [Destroy All Software](http://destroyallsoftware.com).

To follow along you'll need to head to [Gary's dotfiles repo](https://github.com/garybernhardt/dotfiles/) and snag all his cool git scripts and aliases.

For the following examples we'll be analyzing the [twitter/bootstrap repository](https://github.com/twbs/bootstrap) on github.

Let's get to it!

## git-churn

[git-churn](https://github.com/garybernhardt/dotfiles/blob/master/bin/git-churn) is a script that shows the "churn" (modified files) sorted by increasing frequency.

```
# show churn for the whole repository timeline
% git churn

# show churn for specific directories
% git churn app lib

# show churn for a time range
% git churn --since="1 month ago"

# show churn for a specific author
% git churn --author=USERNAME
```

Let's check out our repo and see what the big files are.

```
% git churn
# (snip to the top 20)
77  less/forms.less
77  lib/scaffolding.less
79  bootstrap-1.0.0.min.css
81  docs/assets/css/bootstrap-responsive.css
95  bootstrap-1.0.0.css
97  docs/templates/pages/components.mustache
100 docs/less.html
100 docs/templates/pages/base-css.mustache
104 docs/scaffolding.html
119 lib/forms.less
171 docs/assets/css/docs.css
182 docs/components.html
190 docs/base-css.html
201 lib/patterns.less
245 docs/javascript.html
268 bootstrap.min.css
282 docs/assets/css/bootstrap.css
319 bootstrap.css
336 docs/index.html
461 docs/assets/bootstrap.zip
```

Cool, so we can see that they probably generate that zipfile automatically or at least rebuild it frequently.

Interestingly, the docs get some significant attention. It seems that the docs are not an afterthought in this repo!

Let's see what fat has been up to.

```
% git churn --author=jacobthornton@gmail.com --since='1 month ago'
# (snip a bunch)
5       assets/bootstrap.zip
5       assets/js/bootstrap.js
5       assets/js/bootstrap.min.js
5       docs/assets/css/bootstrap.css
5       docs/assets/js/bootstrap-typeahead.js
6       docs/assets/js/bootstrap-scrollspy.js
6       js/bootstrap-scrollspy.js
8       docs/assets/js/bootstrap.min.js
9       docs/assets/bootstrap.zip
9       docs/assets/js/bootstrap.js
12      Makefile
```

Cool eh? This is really handy when you are joining a new team and want to see what your coworkers have had in the fire.

## git log

So now you've got a shortlist of Interesting Files from git churn, time to see what's been going on with them.

You can pass specific files to `git log` and get the history for just those files. Handy!

```
# show the commits on Makefile since a month ago
% git log --since='1 month ago' -- Makefile

# show the commits with file diffs since a month ago
% git log -p --since='1 month ago' -- Makefile
```

## pretty git log

This is provided in [Gary's .githelpers](https://github.com/garybernhardt/dotfiles/blob/master/.githelpers), but if you want to get a glimpse without some of the niceities you can run this:

```
git log --graph --pretty="tformat:%C(yellow)%h%Creset %Cgreen(%ar)%Creset %C(bold blue)<%an>%Creset %C(red)%d%Creset %s"
```

To get output like this:

![pretty git log](/assets/images/2012-05-11/pretty_git_log.png)

Gorgeous isn't it? The text in red after a commiter's name is the branch. From this listing we can see that master is the most current version of the code and is a few commits ahead of v2.0.3. This is a really handy way to see the state of an entire repo, use it.

## git-divergence

[git-divergence](https://github.com/garybernhardt/dotfiles/blob/master/bin/git-divergence) is another script from Gary. This one shows the incoming/outgoing changes between two commits. Super useful for comparing, say, a feature branch against master (or develop, whatever it was branched from).

Let's checkout those commits between master (which we're currently on) and v2.0.3.

```
# on local master
% git div v2.0.3
```

![git divergence output](/assets/images/2012-05-11/pretty_git_log.png)

With this output we see the commit message and each of the files altered with line-count diffs.

From this quick snapshot we're happy to see that the authors in this repo are making small commits with few changes. Be worried if you see lots of commits that have so many files they scroll off the view and huge line diffs.

## vim-fugitive

The vim-fugitive plugin for vim is chock full of git goodness. Even if you aren't a vim user you might find its blame view helpful.

In vim, with the plugin, in a file that you are interested in analyzing you just run `:Gblame` to get a great view of who's been working on what.

![vim fugitive blame](/assets/images/2012-05-11/vim_fugitive_blame.png)

So we have Mark Otto to blame (credit?) for all the "le comments". :-)

## Recap

Sorry for the whirlwind pace here. I wrote this straight through in one lunch break. But hopefully you've gotten at least a couple tips or tricks, and I've narrowly achieved my deadline of a new post this week. Woot!

## Keep an eye on: Metior

[Metior](https://github.com/koraktor/metior) is a tool that looks like it could be really interesting someday. Currently it's just an API around the git log configured with some interesting, but basic, reports. The "Future plans" have some good bullet points so I'll be keeping an eye on this project.


