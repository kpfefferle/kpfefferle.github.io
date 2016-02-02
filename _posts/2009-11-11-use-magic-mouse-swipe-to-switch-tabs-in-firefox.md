---
layout: post
title: Use Magic Mouse Swipe to Switch Tabs in Firefox
tags:
- apple
- firefox
- How-To
- magic mouse
- swipe
- tabs
status: publish
type: post
published: true
meta:
  _edit_last: "1"
  btc_comment_summary: a:1:{i:0;a:3:{s:11:"comment_src";s:7:"twitter";s:3:"cnt";s:2:"11";s:7:"enabled";s:1:"1";}}
  btc_comment_counts: a:1:{i:0;a:3:{s:11:"comment_src";s:7:"twitter";s:3:"cnt";s:1:"8";s:7:"enabled";s:1:"1";}}
  btc_post: a:6:{s:2:"ID";s:3:"105";s:13:"post_date_gmt";s:19:"2009-11-11 21:31:13";s:23:"initial_import_date_gmt";s:19:"2009-11-11 21:32:35";s:20:"last_import_date_gmt";s:19:"2009-12-01 18:19:18";s:4:"hits";s:2:"11";s:6:"misses";s:3:"163";}
  btc_twitter: "107"
  btc_retweet: "107"
---
I picked up my [Apple Magic Mouse](http://www.apple.com/magicmouse/) a couple weeks ago, and I am totally in love with it. However, most of the programs I regularly use for web development don't take advantage of the two-finger swipe gesture. Luckily, Firefox includes support for gestures that let me set up my mouse to switch tabs with the left/right swipe!

### How to Do It

1. Open Firefox.
2. Type "about:config" into the address bar. After a warning about voiding your warranty (on a free open source product), screwing up your browser, and/or creating a wrinkle in the space/time continuum, this will bring up the Firefox configuration options.
3. Use the filter box to search for "gesture.swipe" and you should see the results below.
4. Double-click on the values for "browser.gesture.swipe.left" and "browser.gesture.swipe.right" and change them to "Browser:PrevTab" and "Browser:NextTab" respectively.
5. Resume browsing as usual and enjoy your new tab-swiping functionality!

![Firefox Settings](/images/Firefox-Magic-Mouse-Screen-Grab.png)

MUCH better than the default back/forward swipe functionality!

**UPDATE (12/14/09):** If you are interested in doing much more with your Mighty Mouse than this, you might want to check out [BetterTouchTool](http://blog.boastr.net/), a utility for adjusting all sorts of gestures for both the MacBook touch pad and the Mighty Mouse (they have good taste in WP theme also)!
