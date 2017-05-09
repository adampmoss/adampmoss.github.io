---
layout: post
title:  "Cookies, Full Page Cache and Private Data Tracking in Magento 2"
permalink: "/magentofox/private-data-tracking-in-magento-2/"
date:   2017-05-8 9:00:00 +0000
categories: magento magento2
excerpt: In this post, Adam Moss explains how to track private data using cookies in Magento 2 without being stung by full page cache.
author: Adam Moss
comments: true
body_class: magento-fox
reading-time: 15 mins
photo: "/assets/featured/gtm-m2.png"
---

Let me start by saying this: learning how to track private data in Magento 2 was a ballache. Now that's out of my system I can continue with this post... Yes it was a ballache but it ended up being very easy. Here's the situation:

There are tons of JavaScript tracking scripts that get added to sites in order to send all kinds of data to their respective systems. They're the Starbucks of the web, a new one seems to pop up every day. By the way, if you're not using [Google Tag Manager](https://www.google.co.uk/analytics/tag-manager/) to do this then you're insane.

Well, in case you haven't guessed this post is all about how to implement GTM tracking in Magento 2 without having your data carved into stone for all to see by Magento 2's full page cache.

### The problem

I had a situation where I had to track a lot of customer data and therefore had to make this data available in the GTM Data Layer.

To keep it simple I'll just focus on how to track a newsletter subscriber's email address when they use the basic newsletter form. This data is not posted back to any page on the website, and even if it was, the name could potentially get cached on the next loaded page meaning the next user could see my email address.

As tracking code normally resides within the page HTML how do you get around this?

### Local storage?

One option could be to save the posted data in local storage and then access it via JS on the next page load (in the same fashion as the mini cart). It then turned out that local storage is basically the last thing loaded so my `datalayer.push()` script had already ran twenty minutes prior with a null value.

### The solution... use cookies!

I looked at the way Magento had handled `datalayer.push()` in their own `Magento_GoogleTagManager` module and it seemed that the most sensible course of action would be to use cookies in the same way.

1. Intercept the data in an observer/plugin
2. Set data to a cookie in PHP
3. Retrieve data on frontend in JS
4. Delete cookie

I have left out the majority of the basic module files for simplicity, but here's the key bits:

#### /etc/di.xml

First I register a [plugin](/magentofox/magento-2-basics-part-6-using-plugins/) that will watch a function in the following class: **Magento\Newsletter\Model\Subscriber**.

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Magento\Newsletter\Model\Subscriber">
        <plugin name="trackSubscriber" type="MagentoFox\Track\Plugin\TrackSubscriber" sortOrder="1" disabled="false"/>
    </type>
</config>
{% endhighlight %}

#### /Plugin/TrackSubscriber.php

I'm using a plugin to set a cookie after a customer has successfully subscribed to the newsletter. Once I have the information I need I use the `CookieManagerInterface` and `CookieMetadataFactory` classes to set my cookie and I'm JSON encoding the result.

{% highlight php %}
<?php
namespace MagentoFox\Track\Plugin;

class TrackSubscriber
{
    protected $catalogSession;
    protected $cookieManager;
    protected $cookieMetadataFactory;
    protected $jsonHelper;

    public function __construct(
        \Magento\Catalog\Model\Session $catalogSession,
        \Magento\Framework\Stdlib\CookieManagerInterface $cookieManager,
        \Magento\Framework\Stdlib\Cookie\CookieMetadataFactory $cookieMetadataFactory,
        \Magento\Framework\Json\Helper\Data $jsonHelper
    ) {
        $this->catalogSession = $catalogSession;
        $this->cookieManager = $cookieManager;
        $this->cookieMetadataFactory = $cookieMetadataFactory;
        $this->jsonHelper = $jsonHelper;
    }

    public function afterSubscribe(\Magento\Newsletter\Model\Subscriber $subscriber)
    {
        if ($subscriber->getStatus()) {

            $subscriberData = $subscriber->getData();

            $publicCookieMetadata = $this->cookieMetadataFactory->createPublicCookieMetadata()
                ->setDuration(3600)
                ->setPath('/')
                ->setHttpOnly(false);

            $customerData = [
                'Email' => $subscriberData['subscriber_email']
            ];

            $this->cookieManager->setPublicCookie("customerData",
                $this->jsonHelper->jsonEncode($customerData),
                $publicCookieMetadata
            );
        }
    }
}
{% endhighlight %}

#### /view/frontend/requirejs-config.js

I'm now declaring a new require class that will contain methods I use for handling cookies:

{% highlight javascript %}
var config = {
    map: {
        '*': {
            MagentoFoxTrack:      'MagentoFox_Track/js/cookies'
        }
    }
};
{% endhighlight %}

#### /view/frontend/web/js/tracking.js

The reason for these functions will become clear in a moment.

{% highlight javascript %}
define([],
    function() {
        "use strict";

        return {

            getCookie : function(name){
                var cookie = " " + document.cookie;
                var search = " " + name + "=";
                var setStr = null;
                var offset = 0;
                var end = 0;
                if (cookie.length > 0) {
                    offset = cookie.indexOf(search);
                    if (offset != -1) {
                        offset += search.length;
                        end = cookie.indexOf(";", offset);
                        if (end == -1) {
                            end = cookie.length;
                        }
                        setStr = decodeURI(cookie.substring(offset, end));
                    }
                }
                return(setStr);
            },

            delCookie : function(name) {
                var date = new Date(0);
                var cookie = name + "=" + "; path=/; expires=" + date.toUTCString();
                document.cookie = cookie;
            },

            parseJson : function (json) {
                var json = decodeURIComponent(json.replace(/\+/g, ' '));
                return JSON.parse(json);
            }
        }
    });
{% endhighlight %}

#### /view/frontend/templates/subscriber.phtml

This file is where the magic happens. First we initialise our new JS class `MagentoFoxTrack` and we pass it as an argument with require. This allows our methods to be used wherever and whenever.

We then grab our `customerData` cookie, check that it is not null, parse the JSON which will turn it back into an object, and push to GTM. FInally we delete the cookie on the fly.

{% highlight html %}
<script type="text/x-magento-init">
{
    "*": {
        "MagentoFoxTrack": {}
    }
}
</script>
<script>
    require([
        'MagentoFox_Track/js/cookies'
    ], function(cookies){

        var customerData = cookies.getCookie("customerData");

        if (customerData !== null) {
            customerData = cookies.parseJson(customerData);
            dataLayer.push(customerData);
            cookies.delCookie("customerData");
        }
    });
</script>
{% endhighlight %}

This may not turn out to be the 100% best way of solving this particular problem, but having thoroughly tested with various types of customer data I have found it to be consistently reliable.

Thanks for reading and please feel free to comment if you have any suggestions or improvements.
