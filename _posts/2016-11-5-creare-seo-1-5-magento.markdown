---
layout: post
title:  "CreareSEO 1.5 for Magento 1"
permalink: "/magentofox/magento-creareseo-1-5/"
date:   2016-11-5 9:00:00 +0000
categories: magento creareseo
author: Adam Moss
comments: true
body_class: magento-fox
reading-time: 5 mins
photo: "magento-fox.png"
---

It's fair to say that CreareSEO has been left on the shelf for quite some time due to change of ownership and other commitments. I was able to dedicate some time to make some important updates over the last month so I'm happy to announce version 1.5 is now available on GitHub: [CreareSEO 1.5](https://github.com/adampmoss/CreareSEO/releases/tag/1.5)

### HTML Sitemap Improvements
The HTML sitemap module has been refactored significantly to reduce server load time. There is now no limit on the depth of the category tree so all categories can be listed in a nested unordered list.

I have also taken the decision to remove the skin/css styling that came bundled in the package because I wanted the styling to be as 'vanilla' as possible, allowing devs to style in their own way if they wish. I understand this may mess up the default styling slightly for some people (sorry!).

![HTML Sitemap Front](/assets/posts/sitemap.png)

Also in this version you can specify your own HTML Sitemap page title, meta description and in-page description.

### Feature: Categories in Default Product Titles & Meta Descriptions

A [long-standing request](https://github.com/adampmoss/CreareSEO/issues/33) is finally here. When creating your default page titles / meta descriptions for products you can now use the following new tags:

- `[first_category]` outputs the first category found belonging to the product
- `[categories]` outputs all categories found belonging to a product in a comma-separated list

For example the following configuration:

![Categories in Meta Data Config](/assets/posts/categories-in-meta.png)

Would output:

{% highlight html %}
<title>Batman T Shirt - Batman Merch</title>
<meta name="description" content="Batman T Shirt - Batman Merch, T Shirts" />
{% endhighlight %}

### CMS Canonical Tag Configuration

The previous logic for setting canonical tags on CMS pages was flawed because it could switch depending on whether you're on a secure page or not. 1.5 allows you to specify in the config whether you want to canonicalise http or https protocols, and also gives you the option to add or remove the trailing slash.

Go to **System > CreareSEO > General Settings**

![CMS Canonical Config](/assets/posts/cms-canonical.png)

### Other Updates

Patch SUPEE-8788 caused a couple of issues with Magento's admin and product page canonical redirects which were swiftly dealt with by the community. A big thanks to [Jeroen Nijhuis](https://github.com/unxsist), [Laura Folco](https://github.com/lfolco) and [Simon Sprankel](https://github.com/sprankhub) for their contributions and support.

- [PR #78](https://github.com/adampmoss/CreareSEO/pull/78)
- [PR #81](https://github.com/adampmoss/CreareSEO/pull/81)

