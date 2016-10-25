---
layout: post
title:  "Magento 2 Basics Part 2 - Creating a Frontend Controller"
permalink: "/magentofox/magento-2-basics-part-2-creating-a-frontend-controller/"
date:   2016-03-16 11:00:00 +0000
categories: magento2 magento
author: Adam Moss
comments: true
body_class: magento-fox
reading-time: 8 mins
photo: "/assets/featured/magento-fox.png"
---

Welcome to part 2 of my mini-series on creating basic modules in Magento 2. In this part I'll show you how to create a simple front controller and output some content.

- [Part 1 - Setting up your module](/magentofox/magento-2-basics-part-1-setting-up-your-module/)
- **Part 2 - Creating a frontend controller**
- [Part 3 - Creating a helper](/magentofox/magento-2-basics-part-3-creating-a-helper/)
- [Part 4 - Creating an observer](/magentofox/magento-2-basics-part-4-creating-an-observer/)
- [Part 5 - Creating an admin page](/magentofox/magento-2-basics-part-5-creating-an-admin-page/)
- [Part 6 - Using plugins](/magentofox/magento-2-basics-part-6-using-plugins/)

Let's get to it then, it's easier than a Sunday morning at Lionel Richie's house.

### Step 1 - Create Magefox/Example/etc/frontend/routes.xml

Add the following code to the file, this is the only config XML needed during this exercise. In the same vein as Magento 1 we are declaring a frontend route using the frontName "magentofox" using the standard router object - this means we can expect to find our frontend path at http://domain.com/magentofox

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="magentofox" frontName="magentofox">
            <module name="Magefox_Example" />
        </route>
    </router>
</config>
{% endhighlight %}

### Step 2 - Create Magefox/Example/Controller/Index/Index.php

Gone is the lowercase 'controllers' directory, gone are the {Something}Controller.php naming conventions and gone are the individual actions within those controllers.

We create a separate class for our individual actions, followed by an execute method for rendering the page content.

{% highlight php %}
<?php

namespace Magefox\Example\Controller\Index;

class Index extends \Magento\Framework\App\Action\Action
{
    public function execute()
    {
        $this->_view->loadLayout();
        $this->_view->renderLayout();
    }
}
{% endhighlight %}

Go to the frontend and you'll see..... a blank screen. We need to tell Magento that there are layout files associated with this page.

### Step 3 - Create Magefox/Example/view/frontend/layout/magentofox_index_index.xml

Yes you read that correctly, 'view' is lowercase, because why not? Hopefully you can appreciate the logic behind the filename specified above as simply a reflection of the controller path.

{% highlight xml %}
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <head>
        <title>Magento Fox</title>
    </head>
</page>
{% endhighlight %}

If you refresh your page on the frontend now you should see the default page layout of your theme with a heading set as "Magento Fox".

Let's add some content to this page by modifying the file as follows:

{% highlight xml %}
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" layout="1column" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <head>
        <title>Magento Fox</title>
    </head>
    <!-- New content below -->
    <body>
        <referenceContainer name="content">
            <block class="Magefox\Example\Block\Example" name="magefox.example"
                   template="magefox/example.phtml" />
        </referenceContainer>
    </body>
</page>
{% endhighlight %}

Hoepfully for seasoned Magento 1 developers this is looking very familiar. We've referenced two files that have not been created yet so let's make them.

### Step 3 Create Magefox\Example\Block\Example.php

In the same fashion as Magento 1 we create a block class within the /Block directory. Remember, I've kept this as stripped down as possible for now just to illustrate the basics.

{% highlight php %}
<?php
namespace Magefox\Example\Block;

use Magento\Framework\View\Element\Template;

class Example extends Template
{
    public function getHello()
    {
        return "Hello World";
    }
}
{% endhighlight %}

### Step 4 - Create Magefox/Example/view/frontend/templates/magefox/example.phtml

This is our template file that will render within the content block list as specified in our layout XML earlier. One difference to note is that `$block` rather than `$this` is used to access the class methods, for example `$block->getHello()`

That's all to this part. You should now have a working module with a frontend controller and a basic block with a template for displaying content.

[Part 3 - Creating a helper &raquo;](/magentofox/magento-2-basics-part-3-creating-a-helper/)