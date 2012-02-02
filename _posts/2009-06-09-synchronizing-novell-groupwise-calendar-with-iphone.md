---
layout: post
title: Synchronizing Novell GroupWise Calendar with iPhone
tags:
- Google Calendar
- GroupWise
- How-To
- iPhone
- synchronize
status: publish
type: post
published: true
meta:
  _edit_last: "1"
  btc_post: a:6:{s:2:"ID";s:2:"73";s:13:"post_date_gmt";s:19:"2009-06-09 18:21:32";s:23:"initial_import_date_gmt";s:19:"2009-06-09 20:36:55";s:20:"last_import_date_gmt";s:19:"2009-07-09 17:13:08";s:4:"hits";s:1:"6";s:6:"misses";s:3:"818";}
  btc_retweet: "22"
  btc_twitter: "22"
  btc_comment_summary: a:1:{i:0;a:3:{s:11:"comment_src";s:7:"twitter";s:3:"cnt";s:1:"6";s:7:"enabled";s:1:"0";}}
  btc_comment_counts: a:0:{}
---
As the Web Manager for a science center, I get a lot of people who come to me at work with random technology questions. For a couple of months, I have had coworkers asking me how they might be able to synchronize their Novell GroupWise calendar at work with their iPhone. I have always just typed in important appointments into the iPhone by hand, but yesterday I did a little Google searching, and I found out that GroupWise to iPhone synching can actually be quite seamless when you use the right middleman: [Google Calendar](http://calendar.google.com/)!

## Get a Google Account

If you don't have one already, sign up for a Google/Gmail account at [gmail.com](http://www.gmail.com/). If you don't use your Gmail account for anything other than this, that's fine. However, your Google account will also be good in the future for logging in to all sorts of other cool Google online services.

## Google Calendar

Now visit [calendar.google.com](http://calendar.google.com/) and log in with your new Gmail account. It asks for a couple additional bits of info to activate your Calendar, and then you will be set to go. The default calendar will be named after your Gmail address, but you can edit the name to something more appropriate like "Work." Also, you can add additional calendars for other types of appointments (I have a "Personal" calendar for myself and a "Family" calendar that I share with my wife in addition to my GroupWise-synced "Work" calendar). Note that when you initially sync your iPhone to Google Calendar, **all existing appointments in your iPhone will be wiped clean**, so take time to fill your Google Calendar with anything you want to keep from your iPhone calendar.

## Delegate GroupWise Appointments to Gmail

Open your desktop GroupWise client (web version won't work) and create a new rule much like you would for an out of office auto-reply. The new rule should do the following: "When event is "Filed Item" to "Calendar" folder, if conditions are "Appointment," the actions are "delegate to (your username)@gmail.com." In the delegate action, type "GWDelegate" in the comments to recipient.

This new rule will automatically send a delegated copy of any appointments you accept in GroupWise to your Gmail address (which is attached to your Google Calendar). We will use the "GWDelegate" comment to keep your Gmail inbox from getting flooded with your appointments in the next step.

## Filter Out (Delete) the Messages to Gmail

Log into Gmail and click "Create a filter" up near the search box. Filter out messages "From: (your work email address)" that "Has the words: GWDelegate" and click "Next Step." On the actions menu, check "Skip the Inbox" and "Delete It." This will send all of your delegated appointments straight to your Trash (they will already be added to your Google Calendar).

## Test the Connection Between GroupWise and Google Calendar

Create a new appointment with yourself (an actual appointment - "Posted Appointments" don't seem to work). Accept it and check to make sure it shows up in your Google Calendar soon after you accept it. You can also check your Gmail Trash for the delegated appointment that was automatically filtered from your Gmail Inbox.

You will need to open all of your existing accepted future appointments and manually delegate them to your Gmail address with the "GWDelegate" comment in the message body to add them to your Google Calendar. The new GroupWise Rule will take care of all future appointments from this point forward.

## Synch iPhone with Google Calendar

Make sure your Google Calendar contains all of the appointments that you will want to have in your iPhone, because **your iPhone calendar will be wiped clean during the initial synch with Google Calendar**. Google has [very clear instructions about synching your iPhone with Google Calendar](http://www.google.com/support/mobile/bin/answer.py?hl=en&amp;answer=138740). You can use Safari on your iPhone to [select which calendars you want to synch with your iPhone](http://www.google.com/support/mobile/bin/answer.py?hl=en&amp;answer=139206) (multiple calendars will show up on the iPhone as different colors, and new events you create on your iPhone can be assigned to any of the calendars!).

## Enjoy Your New Synchronized Life!

Your accepted GroupWise appointments will now show up on your iPhone minutes after you accept them in GroupWise! As an added bonus, any events you manually add in your iPhone will sync back to Google Calendar as well. If you share a calendar (like the "Family" one my wife and I share), then either person can add items to that calendar on their iPhone, and the item will sync to the other person's phone as well!

I still don't know what took me so long to do this! :)
