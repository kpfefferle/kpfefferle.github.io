---
layout: post
title: To Flash or not to Flash?
tags:
- accessibility
- Adobe
- Android
- COSI.org
- Flash
- Google
- graceful degredation
- Home
- iPhone
- JavaScript
- Nitty Gritty
- Target
- YouTube
status: publish
type: post
published: true
meta:
  _edit_last: "2"
  btc_comment_counts: a:0:{}
  btc_comment_summary: a:0:{}
---
It's been almost a week since my last post, but I'm gonna go with the excuse that I was pretty sick the past few days. Aren't you glad that there is a computer screen between us now?

I've been making some good progress on the new redesign that I had mentioned for the [COSI.org](http://www.cosi.org/) home page. One of the issues I am going to have to deal with as we embark on a redesign of [COSI.org](http://www.cosi.org/) is that of web standards including usability and accessibility. I have to consider how those accessing the content by alternative means will see it...those using mobile devices, those wanting to print out pages, and those who can't see the page at all and instead have the computer read the page out loud to them. This issue was brought to public prominence by a [2006 lawsuit brough against Target.com by the National Federation for the Blind](http://news.cnet.com/2100-1030-6038123.html). Their web site was not usable by blind web visitors, and by refusing to change it, they were violating California's Americans with Disabilities Act (the internet equivalent of not having a wheelchair ramp in a building).

So this brings me to my current concern: how, when, and where to make use of Flash on our site. Adobe loves to declare that [Flash content reaches 99% of web users](http://www.adobe.com/products/player_census/flashplayer/). Flash certainly can be used to create some extremely cool interactive experiences like the new ["Got Milk?"](http://gotmilk.com/) campaign. It can be used to entertain like at [Home Star Runner](http://www.homestarrunner.com/). It has even become a superb video delivery tool ([YouTube](http://www.youtube.com/cosiscience) anyone?).

We have used Flash in the past to create some special experiences like many of our [online activities](http://www.cosi.org/visitors/online-activities/). Even a good portion of our in-building exhibit activities are built with Flash. However, such experiences are expensive and time-consuming to create, expensive and time-consuming to update, usually involve some sort of navigational learning curve, and can't easily be read and navigated by those with sight limitations.

The most important blind visitors to any web site are search engines like Google which use automated web "spiders" to catalog site content. Although some search engines have made advances in being able to "read" Flash content, they are still unable to link to the specific point in the Flash movie where that content occurs since the entire Flash movie uses the same browser URL!

Next comes the concept of "graceful degredation." In short, this means that when a visitor to your site has limits to viewing the content you provide (no Flash, no JavaScript, no CSS, or even no images), they will still be able to understand and navigate the content you are presenting. Flash does not degrade well at all...it tends to either work or not. I have become especially sensitive to this since I purchased an iPhone, which displays web content in all of it's desktop browser glory, but does not support Flash. When I visit the current [COSI.org home page](http://www.cosi.org/) on my iPhone, I get a big white block with an error icon where the billboard rotator should be (which occupies almost half of the page). Early news is that Google's new Android cell phone operating system won't support Flash web content either.

So what do I do instead? Well, I think that the new home page is going to feature headlining content using a similar home rotator, but one programmed using JavaScript! No Flash? No problem (even on my iPhone). No JavaScript? You may not get the rotating images, but at least you'll see the first one!

I'll keep you up to date as we continue to develop the new home page. Hopefully it will set the bar pretty high in both design AND content so the rest of the site can follow soon after!
