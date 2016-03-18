---
layout: post
title:  "Magento 2 Basics Part 6 - Using Plugins"
permalink: "/magentofox/magento-2-basics-part-6-using-plugins/"
date:   2016-03-16 14:00:00 +0000
categories: magento2 magento
author: Adam Moss
---

Plugins are a fantastic way of extending Magento 2 without the need for class rewrites by allowing you to modify the input and output of (most) methods belonging to the Magento 2 framework. For those familiar with Wordpress development the theory is similar to the action/hook format.

- [Part 1 - Setting up your module](/magentofox/magento-2-basics-part-1-setting-up-your-module/)
- [Part 2 - Creating a frontend controller](/magentofox/magento-2-basics-part-2-creating-a-frontend-controller/)
- [Part 3 - Creating a helper](/magentofox/magento-2-basics-part-3-creating-a-helper/)
- [Part 4 - Creating an observer](/magentofox/magento-2-basics-part-4-creating-an-observer/)
- [Part 5 - Creating an admin page](/magentofox/magento-2-basics-part-5-creating-an-admin-page/)
- **Part 6 - Using plugins**

In this tutorial I'm going to take a method and show you how it can be modified using the three plugin types:

- before
- after
- around

### Step 1 - Create Magefox/Example/etc/di.xml

In the example below I have declared two classes I want to "observe" with my plugins, and within each I have entered a plugin entry that specifies the class that I want to use in the "type" attribute.

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Magento\Theme\Block\Html\Breadcrumbs">
        <plugin name="changeBreadcrumbs" type="Magefox\Example\Plugin\BreadcrumbsPlugin" sortOrder="1" disabled="false"/>
    </type>
    <type name="Magento\Cms\Model\Page">
        <plugin name="changePage" type="Magefox\Example\Plugin\PagePlugin" sortOrder="1" disabled="false"/>
    </type>
</config>
{% endhighlight %}

The naming convention I've used for my plugin means that I can identify quickly that my plugin is concerned with a page model. I don't know for sure if this is the suggested naming convention but it is in line with Magento's own dev docs from what I've seen.

A couple of important points:

- First argument must always be the observed class (e.g. **Magento\Theme\Block\Html\Breadcrumbs**) - this gives us access to all the parent classes methods.
- The following arguments are the arguments passed by default into the method
- Plugins can be prioritised using the sortOrder attribute. They are ran in ascending order.

Now the file where the magic happens. Plugin classes do not extend or interface any other classes, though they must still use dependency injection if you want to access other objects.

#### Before - Magefox\Example\Plugin\BreadcrumbsPlugin.php

"Before" allows you to modify the arguments of a chosen method, which obviously won't do much if the method has no arguments! Let's take a look at the following example:

{% highlight php %}
<?php
namespace Magefox\Example\Plugin;

class BreadcrumbsPlugin
{
    public function beforeAddCrumb(\Magento\Theme\Block\Html\Breadcrumbs $subject, $crumbName, $crumbInfo)
    {
        if ($crumbInfo['label'] == "Privacy and Cookie Policy")
        {
            $crumbInfo['label'] = "Something else";
        }

        return [$crumbName, $crumbInfo];
    }
}
{% endhighlight %}

In this example I'm taking the two arguments $crumbName and $crumbInfo and I'm checking if the label is "Privacy" then change it to literally something else. Then I'm returning the arguments in the exact correct format at the end of the function. Real simple eh?

#### After - Magefox\Example\Plugin\PagePlugin.php

The "after" option allows you to modify the data that is returned by the method in question. I've chosen a different class this time because the Breadcrumbs class isn't the best example of how this works.

{% highlight php %}
<?php
namespace Magefox\Example\Plugin;

class PagePlugin
{

    public function afterGetContentHeading(\Magento\Cms\Model\Page $page, $contentHeading)
    {
        return "This is the ".$contentHeading;
    }
}
{% endhighlight %}

The second argument is always the value returned by the original method, we simply take that result and do what we want with it. In this case I'm just prefixing some text to the string that is returned.

#### Around - Magefox\Example\Plugin\BreadcrumbsPlugin.php

Our "around" method takes at least two arguments:

1. `$subject` The class being listened two
2. `$proceed` The next plugin or method in line based on sortOrder
3. The rest of the arguments based on the methods

We then have the opportunity within our method to modify the arguments AND change the returned value. Let's go back to our breadcrumbs class:

{% highlight php %}
<?php
namespace Magefox\Example\Plugin;

class BreadcrumbsPlugin
{
    public function aroundAddCrumb(\Magento\Theme\Block\Html\Breadcrumbs $subject, \Closure $proceed, $crumbName, $crumbInfo)
    {
        if ($crumbInfo['label'] == "Privacy and Cookie Policy")
        {
            $crumbInfo['label'] = "Something else";
        }

        $result = $proceed($crumbName, $crumbInfo);

        /* I now that the opportunity to influence what is returned by our method. */

        return $result;
    }
}
{% endhighlight %}

I start by doing my thing to the arguments as per I previous example, and then I've highlighted the area where I have the opportunity to change the result. As the method returns an instance of the class there's not much more I can do to illustrate this.

Plugins are a major step forward for Magento 2 so I hope you enjoy using them!
