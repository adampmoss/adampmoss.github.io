---
layout: post
title:  "Magento 2 Basics Part 1 - Setting Up Your Module"
permalink: "/magentofox/magento-2-basics-part-1-setting-up-your-module/"
date:   2016-03-16 10:00:00 +0000
categories: magento2 magento
author: Adam Moss
comments: true
body_class: magento-fox
reading-time: 5 mins
photo: "/assets/featured/magento-fox.png"
---

In the following tutorials I'm going to take you through the very basics of creating a simple, local Magento 2 module. The series will cover the following topics:

- **Part 1 - Setting up your module**
- [Part 2 - Creating a frontend controller](/magentofox/magento-2-basics-part-2-creating-a-frontend-controller/)
- [Part 3 - Creating a helper](/magentofox/magento-2-basics-part-3-creating-a-helper/)
- [Part 4 - Creating an observer](/magentofox/magento-2-basics-part-4-creating-an-observer/)
- [Part 5 - Creating an admin page](/magentofox/magento-2-basics-part-5-creating-an-admin-page/)
- [Part 6 - Using plugins](/magentofox/magento-2-basics-part-6-using-plugins/)

*Note - I'm going to cover the basics of a local module created in app/code. Modules should ideally be developed in separate repositories and installed with Composer.*

Let's start by clearing up exactly what a module is. Magento defines a module as the following:

>A module is basically a package of code that encapsulates a particular business feature or set of features.

Magento 2 encourages modules to take responsibility for their own separate features and to explicity declare any dependancies (if any).  The module should be working code in its own right and if possible be unaware of the system it is being plugged into.

With that in mind, let's go through the steps of creating our own module.

### Step 1 - Decide on Namespace (Vendor) and Module Name

Just like in Magento 1 a module must belong to a namespace, which is now referred to as a vendor:

- **Vendor:** Magefox
- **Module:** Example

With this in mind create the following directory: **app/code/Magefox/Example**

### Step 2 - Create Magefox/Example/etc/module.xml

This file is basically the same as your module declaration file in Magento 1 (the one in app/etc/modules), except it is now conveniently stored within your module. Everything in one place!

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
    <module name="Magefox_Example" setup_version="1.0.0">
    </module>
</config>
{% endhighlight %}

### Step 3 - Create Magefox/Example/registration.php

This file was introduced shortly before GA was released and simply serves as a way to register your module as a component with Magento. Simply add your vendor/module name as the second argument.

{% highlight php %}
<?php
\Magento\Framework\Component\ComponentRegistrar::register(
    \Magento\Framework\Component\ComponentRegistrar::MODULE,
    'Magefox_Example',
    __DIR__
);
{% endhighlight %}

All that's left for us to do is to activate our module, so using command lin navigate to your Magento 2 root directory and run the following command:

    php bin/magento setup:upgrade

Congratulations you've just created a Magento 2 module! If you want to check it has installed properly login to the admin and go to **Stores > Configuration > Advanced > Advanced** and you should see it in the "Disable Modules Output" section.

[Part 2 - Creating a frontend controller &raquo;](/magentofox/magento-2-basics-part-2-creating-a-frontend-controller/)