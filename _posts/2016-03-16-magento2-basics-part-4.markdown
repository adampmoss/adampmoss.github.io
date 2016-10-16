---
layout: post
title:  "Magento 2 Basics Part 4 - Creating an Observer"
permalink: "/magentofox/magento-2-basics-part-4-creating-an-observer/"
date:   2016-03-16 13:00:00 +0000
categories: magento2 magento
author: Adam Moss
comments: true
body_class: magento-fox
---

Part 4 is another pretty straighforward section, how to add an event observer in Magento 2. Yes event-dispatch is still present in Magento even with the introduction of plugins (which I'll come onto at some point).

- [Part 1 - Setting up your module](/magentofox/magento-2-basics-part-1-setting-up-your-module/)
- [Part 2 - Creating a frontend controller](/magentofox/magento-2-basics-part-2-creating-a-frontend-controller/)
- [Part 3 - Creating a helper](/magentofox/magento-2-basics-part-3-creating-a-helper/)
- **Part 4 - Creating an observer**
- [Part 5 - Creating an admin page](/magentofox/magento-2-basics-part-5-creating-an-admin-page/)
- [Part 6 - Using plugins](/magentofox/magento-2-basics-part-6-using-plugins/)

There are two steps to creating an observer once you've decided which event you want to hook into. First you declare it in a brand-spanking new XML file and then you create the observer class, a separate class for each observer!

### Step 1 - Create Magefox/Example/etc/frontend/routes.xml

In this example we're dealing with a frontend event (meaning that the same event fired in the admin would have no implications).

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Event/etc/events.xsd">
    <event name="controller_action_postdispatch">
        <observer name="magefox.fire" instance="Magefox\Example\Observer\Fire" />
    </event>
</config>
{% endhighlight %}

In the example above we've simply created a new hook to look out for the execution of **controller_action_postdispatch** on the frontend - a fairly common occurance I would imagine! Then we've told it to run the script in our observer, which we'll now create below.

### Step 2 - Creare Magefox/Example/Observer/Fire.php

This is the most simple of observers, no dependencies, just an execute method like those found in Magento 2 controllers.

{% highlight php %}
<?php
namespace Magefox\Example\Observer;

use Magento\Framework\Event\ObserverInterface;

class Fire implements ObserverInterface
{
    /**
     * Test observer to echo "Done"
     *
     * @param \Magento\Framework\Event\Observer $observer
     *
     */
    public function execute(\Magento\Framework\Event\Observer $observer)
    {
        echo "Done";
    }
}
{% endhighlight %}

"Done" will now be echoed annoyingly at the top of the page when the event is dispatched. As with Magento 1 observer data can be accessed with `$observer->getEvent()`

That's all folks!

[Part 5 - Creating an admin page &raquo;](/magentofox/magento-2-basics-part-5-creating-an-admin-page/)