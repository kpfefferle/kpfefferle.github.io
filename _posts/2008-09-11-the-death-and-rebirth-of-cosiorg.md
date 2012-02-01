---
layout: post
title: The Death and Rebirth of COSI.org
tags:
- Adapt
- COSI.org
- Google Custom Search
- Integrate
- Lost Egypt
- Nitty Gritty
- Parents Magazine
- PHP
- site crash
- technology
- Time Warner
- web server
- WordPress
status: publish
type: post
published: true
meta:
  _edit_last: "1"
  btc_comment_counts: a:0:{}
  btc_comment_summary: a:0:{}
---
Three weeks ago, [COSI.org](http://www.cosi.org/) had one of its darkest days. At 1:30 PM on Tuesday, August 19th, the [COSI web site](http://www.cosi.org/) died an unexpected death. The timing couldn't have been worse, as we were celebrating our [recent #1 ranking by Parents Magazine](http://www.cosi.org/press-room/press-releases/?year=2008&date=1218600000&id=0) with a special $1 admission offer that day. Radio and TV outlets throughout Columbus were directing listeners and viewers to [www.cosi.org](http://www.cosi.org/) for more information about the promotion, and the site wasn't there to be seen!

This situation really illustrates how fragile technology can be. We take for granted the millions upon millions of electrical signals that are sent back and forth inside a computer that let it think and process our applications. We also take for granted the code of the applications that tell the computer how to logically process the information it is given. Like the thousands of little explosions that happen within our automobile engines every second, we usually don't appreciate such details until they stop working.

For the past few years, the [COSI web site](http://www.cosi.org/) had been hosted within the COSI building on a Windows server set up and maintained by our IT department. They have always done a wonderful job at handling the server maintenance, keeping it connected to the internet, keeping the server secure, and managing our domain names. Our site is powered by the Adapt CMS (Content Management System) developed by [Integrate Inc.](http://www.integrateinc.com/) Adapt is programmed in the PHP language, and a standard Windows web server does not natively run PHP. To fill this gap, [Integrate](http://www.integrateinc.com/) had installed a PHP application on the web server to process the PHP code. This PHP application is the piece that unexpectedly failed and brought the whole site crumbling down.

Somewhat ironically, I had submitted a proposal to our VP of Advancement *the day before the site crash* that suggested we move the site to an outside web host. Doing so would give us greater flexibility within the server, allow for more interaction with our site users while still maintaining security, and also provide us with a native PHP framework to run on. In fact, making the move would save us hundreds of dollars a year in hardware and software costs AND save our IT team's valuable time. The proposal seemed like the best of both worlds with enhanced service for a lower cost, and was just awaiting final approval from our COSI leadership.

With the site down, my proposal jumped right to the top of COSI's priority list! The COSI leadership readily approved the plan, and I had [1&1 Internet](http://www.1and1.com/) set up a dedicated server for COSI that day. I had my computer uploading all of the files to the new server throughout Wednesday morning, and [Integrate](http://www.integrateinc.com/) was kind enough to send their developer Brian over to help me move the databases and chase down the bugs. Brian and I ended up being here until 1:30 AM Thursday morning ironing out all the wrinkles, but we finally had the site up and running on the new server! Our IT department was working with Time Warner to make the final switch happen...directing the web traffic for [www.cosi.org](http://www.cosi.org/) to the new server.

Around 11:00 AM on Thursday, August 21, the new domain name settings took effect and [COSI.org](http://www.cosi.org/) was back up and running! The migration of our site from an internal server to an external one (a project I had planned to do over the course of several days) had been successfully completed in under 48 hours! A couple additional bugs cropped up after we were live regarding our contact and registration forms and the site search. The email forms were working again within a day, and I eventually solved the site search problems by upgrading from the older htdig search we were using to [Google's Custom Search](http://www.google.com/cse) tool that not only restored the function to the site but also improved the quality of the search results!

In all, those few fateful days in the middle of August were an interesting experience to say the least. In the end, we got the site back up, relieved some of the load that had been placed on our IT team (who has plenty of hard work to do without having to manage a web server), increased our site's flexibility for the future, and even upgraded a few features along the way!

Since the server move, I have been able to use the new dedicated server to create a complete development copy of [COSI.org](http://www.cosi.org/) where I can test out new ideas and work on projects without risking damage to the live site. In addition, we can set up entire web applications (like this [WordPress](http://www.wordpress.org/) blog you are reading) right on our own server. As we prepare to open our new [Lost Egypt](http://lostegypt.wordpress.com/) traveling exhibition next summer, we will even be able to create a standalone web site dedicated to the exhibit using our new dedicated server!

The possibilities don't end there.  What would you like to see us do?
