---
layout: post
title:  "Magento 1 - Registration Form Not Working"
permalink: "/magentofox/magento-registration-form-not-working/"
date:   2016-10-24 18:00:00 +0000
categories: magento
author: Adam Moss
comments: true
body_class: magento-fox
reading-time: 4 mins
---

I'm gonna try and get straight to the point with the post because this actually quite an old problem, but I it recently came to my attention just how un-documented the solution that worked for me was.

### What's the bug?

You try and register or login as a customer using your website's frontend and the page simply refreshes without an error message or even a damn apology. The bug is best known for its presence in Magento CE 1.9.2.2, EE 1.14.2.3 (I think) and the wonderful patch SUPEE-6788.

The solution that works for a lot of people is already very well documented here so I'll not bother repeating what has already been said: [https://www.dudesquare.nl/blog/2015/11/02/customer-registration-not-working-magento-1-9-2-2/](https://www.dudesquare.nl/blog/2015/11/02/customer-registration-not-working-magento-1-9-2-2/)

In a nutshell it's to do with hidden form keys not existing in frontend forms.

### That didn't work for me! What else you got??

So you have all your form keys in place and the error persists, and you now have about as much hair as Stone Cold Steve Austin. The issue (for me at least) ended up being to do with session cookies messing up the form keys.

Using Xdebug I could see that the form key that gets sent by the registration form was failing its vaildation check because by the time the createPostAction was called a new form key had been created!

What caused this to happen? It turned out that somehow our site had created two `frontend` session cookies and a different one was being used either side of the form POST. Helpful eh?

#### To fix this there are two solutions...

1) If your site is fairly stable you can delete the duplicate `frontend` session cookie. Ideally the remaining cookie will be something like: **yourdomain.com**

2) Alternatively, go to **System > Configuration > General > Web** and add your domain URL to the Cookie Domain field as shown below. Credit: [Phil Birnie on Stack Overflow](http://stackoverflow.com/a/37955615/750537)

![Set Cookie Domain](/assets/posts/cookie-domain.png)

This will guarantee a single domain for your frontend cookies meaning that the other will simply be ignored.