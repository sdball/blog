---
layout: single
title: "Add Clicky Web Analytics to Octopress"
date: 2012-02-05 09:39
comments: true
categories: software
---

Octopress is fantastic out of the box. But everyone knows we programmers are
always wanting more. Luckily, the fantasticalness of Octopress includes easy
addition of custom code. Here's how to add Clicky Web Analytics tracking.

<!--more-->

If you have websites. If you care about web analytics. If you haven't already. You need to check out [Clicky Web Analytics](http://getclicky.com).

Octopress supports Google Analytics with just a tracking id in the configuration
files. We *could* add in another site variable and some clicky analytics code
using that variable or we could abstract things out into an even more generic
analytics include file. Neat idea, but [generic analytics in
Octopress](https://github.com/imathis/octopress/pull/177) have already been
addressed. The answer was, pragmatically, to add an even more generic custom
footer area.

Let's edit that area.

In your favorite text editor, open up
`/source/_includes/custom/after_footer.html` in your octopress repo.

Bam! Add whatever wacky code you want to add here. Clicky, woopra,
Piwik&hellip;whatever.

Just copy your clicky tracking code and paste it in here. Now the next time you
do the ol' generate and deploy you'll be getting all those awesome clicky web
analytics.
