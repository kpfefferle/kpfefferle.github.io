---
layout: post
title: "Password Security (For Your Own Good)"
category: 
tags: []
---
{% include JB/setup %}

My time today at [The Brandery][1] was spent putting together the core of Receept’s signup/login functionality (a vitally important part of any web app), and I took a little time to implement a solution for what has always been one of my pet peeves.

When you sign up for any web service, choosing a good password is one of the most important early decisions you make. You want to pick something that will keep your information safe, but safe passwords are often also hard to remember. I personally use [1Password][2] to manage complex passwords that I would never be able to remember myself, but I really do understand why so many people use something that they can easily remember. However, there are some passwords that cross a line from simple and rememberable into just plain unsafe. When the web service RockYou had all their user passwords revealed in December 2009 (in plain text no less), [an analysis of the most popular passwords used on the service produced some pretty depressing results][3]. The top 10 most used passwords on RockYou were:

1. 123456
2. 12345
3. 123456789
4. password
5. iloveyou
6. princess
7. rockyou (the name of the service!)
8. 1234567
9. 12345678
10. abc123

These passwords make me cringe, as it would take a hacker no time at all to compromise your account and all of the data you have accumulated. So guess what? These passwords (and almost 500 others that have been identified as overly common) aren’t able to be used on Receept. We will continue to review our list of too-common passwords, and it will likely grow as we do.

If you attempt to set your password to something that we have identified as too common, you will be kindly prompted to choose something else. Consider it a kind nudge from Receept to improve your safety and security online :)

[1]: http://brandery.org/
[2]: http://agilebits.com/products/1Password
[3]: http://www.imperva.com/docs/WP_Consumer_Password_Worst_Practices.pdf
