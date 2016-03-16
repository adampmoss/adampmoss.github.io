---
layout: post
title:  "Magento 2 Basics Part 3 - Creating a Helper"
permalink: "/magentofox/magento-2-basics-part-3-creating-a-helper/"
date:   2016-03-16 12:00:00 +0000
categories: magento2 magento
author: Adam Moss
---

We have finally arrived at part 3 of my mini-series on creating basic modules in Magento 2. This is a short one - how to create and utilise helpers in Magento 2.

- [Part 1 - Setting up your module](/magentofox/magento-2-basics-part-1-setting-up-your-module/)
- [Part 2 - Creating a frontend controller](/magentofox/magento-2-basics-part-2-creating-a-frontend-controller/)
- **Part 3 - Creating a helper**
- [Part 4 - Creating an observer](/magentofox/magento-2-basics-part-4-creating-an-observer/)
- [Part 5 - Creating an admin page](/magentofox/magento-2-basics-part-5-creating-an-admin-page/)

Helpers have the exact same use in Magento 2 as they do in 1. They are designed for features and functionality that are not specific to a model or block class. Let's begin by making our helper file:

### Step 1 - Create Magefox/Example/Helper/Data.php

Create your helper file and add the code below. I've made a simple method that converts characters of a string to uppercase.

{% highlight php %}
<?php
namespace Magefox\Example\Helper;

class Data extends \Magento\Framework\App\Helper\AbstractHelper
{
    public function upperString($string)
    {
        return strtoupper($string);
    }
}
{% endhighlight %}

### Step 2 - Utilise Helper in our Block (Magefox\Example\Block\Example.php)

In part 2 we created a block specified above, so let's say that we want to access this helper within our block class. By default we will now be introduced to the basics of dependency injection.

{% highlight php %}
<?php
namespace Magefox\Example\Block;

use Magento\Framework\View\Element\Template;

class Example extends Template
{
    /**
    * @var \Magefox\Example\Helper\Data
    */
    protected $_foxyHelper;

    /**
     * @param \Magento\Framework\View\Element\Template\Context $context
     * @param \Magefox\Example\Helper\Data $foxyHelper
     * @param array $data
     */
    public function __construct(
        \Magento\Framework\View\Element\Template\Context $context,
        \Magefox\Example\Helper\Data $foxyHelper,
        array $data = []
    )
    {
        $this->_foxyHelper = $foxyHelper;
        parent::__construct($context, $data);
    }

    public function getHello()
    {
        return $this->_foxyHelper->upperString("Hello world, it's Magento Fox!");
    }
}
{% endhighlight %}

Suddenly things look a little more complicated don't they? Don't panic though because what's going on here is actually pretty simple.

We've declared a new class property **$_foxyHelper** to act as our access to the helper, and we have set this property in our construct by passing in the class path as an argument.

What about the other argument "context"? Well, in order to use the template class's construct we have pass in a context as the parent constructor expects it. It basically tells Magento what type of object the class represents and what it's specific dependencies are. This is always needed when using a custom-built block class's constructor.

That about ties up this tutorial... part 4 awaits.

[Part 4 - Creating an observer &raquo;](/magentofox/magento-2-basics-part-4-creating-an-observer/)