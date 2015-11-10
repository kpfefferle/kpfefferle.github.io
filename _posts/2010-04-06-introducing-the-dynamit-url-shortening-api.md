---
layout: post
title: Introducing the dynamIt URL-shortening API
tags:
- API
- Bit.ly
- How-To
- Nitty Gritty
- Tweetie
- Twitter
- URL
status: publish
type: post
published: true
meta:
  _edit_last: "1"
  btc_comment_summary: a:0:{}
  btc_comment_counts: a:0:{}
  btc_post: a:6:{s:2:"ID";s:3:"120";s:13:"post_date_gmt";s:19:"2010-04-06 17:04:42";s:23:"initial_import_date_gmt";s:19:"2010-04-06 17:07:28";s:20:"last_import_date_gmt";s:19:"0000-00-00 00:00:00";s:4:"hits";s:1:"0";s:6:"misses";s:1:"0";}
---

While using the [Tweetie 2 Twitter client](http://www.atebits.com/tweetie-iphone/) on my iPhone a couple of weeks ago, I realized the app developer saw fit to include the option of using any URL shortening service I might desire if the shortening service provided an appropriate <abbr title="Application Programming Interface">API</abbr>.

<abbr title="Application Programming Interface">API</abbr> stands for [Application Programming Interface](http://en.wikipedia.org/wiki/Application_programming_interface). An <abbr title="Application Programming Interface">API</abbr> allows software to interact with other software. Twitter clients use the [Twitter API](http://apiwiki.twitter.com/) to implement features of [Twitter](http://twitter.com/) and access tweets, Twitter uses the [Bit.ly API](http://code.google.com/p/bitly-api/wiki/ApiDocumentation) to shorten links using the [Bit.ly](http://bit.ly/) service, and we have made creative use of various <abbr title="Application Programming Interface">API</abbr>s on sites like [Columbus College of Art and Design](http://www.ccad.edu/) (which uses the [Google Calendar API](http://code.google.com/apis/calendar/) to power its deep set of [event and news listings](http://www.ccad.edu/calendar/display?q=ccad-events)).

Since [dynamIt already has a URL shortener built into our site](http://www.dynamit.us/blog/2009/03/a-smaller-url/), all we needed was a suitable <abbr title="Application Programming Interface">API</abbr>. I was able to use the existing scripting to generate and store the shortened URL, and only needed to adjust how the shortened URL was returned for suitable <abbr title="Application Programming Interface">API</abbr> use.

[TinyURL](http://tinyurl.com/) has set a precedent for the simplest response possible—just the new shortened URL. Try calling the following action in your browser where [URL] is the URL you would like to shorten:

    http://dynamit.us/url/api.dT?url=[URL]

The response will be the resulting shortened dynamit.us URL in plain text. You can optionally include the title of the page whose URL you are shortening as well:

    http://dynamit.us/url/api.dT?title=[TITLE]&amp;url=[URL]

While this simple plain text response can be interpreted by most applications, some developers may prefer a Bit.ly / JSON style response that looks something like this:

    { "shortUrl": "http://dynamit.us/222" }

If so, just add the URL variable "json=true" to the <abbr title="Application Programming Interface">API</abbr> call (with or without the optional [TITLE] variable):

    http://dynamit.us/url/api.dT?json=true&amp;url=[URL]

The Tweetie 2 application I use on my iPhone will accept either format. In Tweetie 2, go to Settings &gt; Services &gt; URL Shortening &gt; Custom and enter:

    http://dynamit.us/url/api.dT?url=%@

Or if you want to use the Bit.ly / JSON format just for fun:

    http://dynamit.us/url/api.dT?json=true&amp;url=%@

![Tweetie URL Settings](/assets/images/tweetie-url-1.jpg) ![Tweetie URL API](/assets/images/tweetie-url-2.jpg)

Tweetie recognizes "%@" as where it should include the URL in the <abbr title="Application Programming Interface">API</abbr> call. Now whenever I am tweeting from my phone and want to include a URL, Tweetie will automatically use the dynamit.us URL shortener to shorten the link for me!

![Test Tweet](/assets/images/tweetie-url-3.jpg)

Are there other places where you might like to use our URL shortener?
