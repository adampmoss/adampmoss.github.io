---
layout: post
title:  "Magento 2 Basics Part 5 - Creating an Admin Page"
permalink: "/magentofox/magento-2-basics-part-5-creating-an-admin-page/"
date:   2016-03-16 14:00:00 +0000
categories: magento2 magento
author: Adam Moss
comments: true
---

Adding elements to the admin area is always a requirement when working with custom modules as you need to let your user content-manage and configure their functionality.

- [Part 1 - Setting up your module](/magentofox/magento-2-basics-part-1-setting-up-your-module/)
- [Part 2 - Creating a frontend controller](/magentofox/magento-2-basics-part-2-creating-a-frontend-controller/)
- [Part 3 - Creating a helper](/magentofox/magento-2-basics-part-3-creating-a-helper/)
- [Part 4 - Creating an observer](/magentofox/magento-2-basics-part-4-creating-an-observer/)
- **Part 5 - Creating an admin page**
- [Part 6 - Using plugins](/magentofox/magento-2-basics-part-6-using-plugins/)

Adding sections to the admin area is in some ways very different to Magento 1. In this tutorial I'm not going to cover admin grids, instead I'm going to focus on creating a grid-less admin page and adding to the configuration.

## Configuration

The Magento 2 configuration is now found in **Stores > Settings > Configuration**, however a familiar file is needed to add to this:

### Step 1 - Create Magefox/Example/etc/adminhtml/system.xml

In the example below I have created a new tab called "Magento Fox" with a section called "Settings" underneath. There is then a option group called "Settings" with a single Yes/No field called "Show Something?".

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <tab id="magefox" translate="label" sortOrder="200">
            <label>Magento Fox</label>
        </tab>
        <section id="magefox" translate="label" type="text" sortOrder="110" showInDefault="1" showInWebsite="1"
                 showInStore="1">
            <label>Settings</label>
            <tab>magefox</tab>
            <resource>Magefox_Example::settings</resource>
            <group id="settings" translate="label" type="text" sortOrder="120" showInDefault="1" showInWebsite="1"
                   showInStore="1">
                <label>Settings</label>
                <field id="showsomething" translate="label" type="select" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
                    <label>Show something?</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
            </group>
        </section>
    </system>
</config>
{% endhighlight %}

There are obviously tonnes more options you can choose from here - my suggestion is to look at the core modules system.xml to find the code you need for each field type. The options are basically the same as Magento 1 system.xml.

### Step 2 - Create Magefox/Example/etc/config.xml

Config.xml is back, except this time is for setting default configuration values only. At least it's a lot more readable this time round!

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Store:etc/config.xsd">
    <default>
        <magefox>
            <settings>
                <showsomething>1</showsomething>
            </settings>
        </magefox>
    </default>
</config>
{% endhighlight %}

## Admin Pages

Creating an actual page in the admin requires a bit more work because (like the frontend) we have to create a controller using the admin route object and we have to build blocks to make the content. Grids & forms aside, this is how you do it:

### Step 1 - Create Magefox/Example/etc/adminhtml/routes.xml

First we create our admin route as I explained above (more XML!):

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="admin">
        <route id="magefox" frontName="magefox">
            <module name="Magefox_Example" />
        </route>
    </router>
</config>
{% endhighlight %}

### Step 2 - Create Magefox/Example/etc/adminhtml/menu.xml

Now you have to decide how you want people to be able to access your page by creating a conveniently named "menu.xml" file within etc/adminhtml.

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Backend:etc/menu.xsd">
    <menu>
        <add id="Magefox_Example::magefox_stores" title="Magento Fox" module="Magefox_Example" sortOrder="50" parent="Magento_Backend::stores" resource="Magefox_Example::magefox_stores" />
        <add id="Magefox_Example::test" title="Test" module="Magefox_Example" sortOrder="10" parent="Magefox_Example::magefox_stores" action="magefox/foxypage" resource="Magefox_Example::test"/>
        <add id="Magefox_Example::settings" title="Settings" module="Magefox_Example" sortOrder="20" parent="Magefox_Example::magefox_stores" action="adminhtml/system_config/edit/section/magefox" resource="Magefox_Example::settings"/>
    </menu>
</config>
{% endhighlight %}

In the example above I have added a new section to the "Stores" tab which is referenced in the first &lt;add&gt; node where it says **parent="Magento_Backend::stores"**. Underneath this I have then added a link to my new page which will be located at magefox/foxypage, and I've also linked to our new configuration page we created above.

### Step 3 - Create Magefox/Example/etc/acl.xml

We need to ensure that our admin page can be restricted to certain users only. This can be done by making it available to the ACL object.

{% highlight xml %}
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Acl/etc/acl.xsd">
    <acl>
        <resources>
            <resource id="Magento_Backend::admin">
                <resource id="Magento_Backend::stores">
                    <resource id="Magefox_Example::magefox_stores" title="Magento Fox" sortOrder="10" >
                        <resource id="Magefox_Example::test" title="Test" sortOrder="10"/>
                        <resource id="Magefox_Example::settings" title="Settings" sortOrder="20"/>
                    </resource>
                </resource>
            </resource>
        </resources>
    </acl>
</config>
{% endhighlight %}

### Step 4 - Create Magefox/Example/Controller/adminhtml/Foxypage/Index.php

We create our admin controller action in pretty much the same way as the frontend action, an execute method renders the page. An additional method here will also check that the user has the sufficient priveledges that we assigned above using ACL.

{% highlight php %}
<?php
namespace Magefox\Example\Controller\Adminhtml\Foxypage;

class Index extends \Magento\Backend\App\Action
{
    /**
     * Check if user has enough privileges
     *
     * @return bool
     */
    protected function _isAllowed()
    {
        return $this->_authorization->isAllowed('Magefox_Example::foxypage');
    }

    /**
     * @return void
     */
    public function execute()
    {
        $this->_view->loadLayout();

        $this->_setActiveMenu('Magefox_Example::foxypage');
        $this->_view->getPage()->getConfig()->getTitle()->prepend(__('Foxy Admin Page'));

        $this->_addBreadcrumb(__('Stores'), __('Stores'));
        $this->_addBreadcrumb(__('Foxy Admin Page'), __('Foxy Admin Page'));

        $this->_view->renderLayout();
    }
}
{% endhighlight %}

### Step 5 - Create Magefox/Example/view/adminhtml/layout/magefox_foxypage_index.xml

Just as we did with the front controller layout XML we do the same with the admin. All I'm doing here is adding a custom block to the content block:

{% highlight xml %}
<page xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_configuration.xsd">
    <body>
        <referenceContainer name="content">
            <block name="hello.admin" class="\Magefox\Example\Block\Adminhtml\Hello"
                   template="hello.phtml" />
        </referenceContainer>
    </body>
</page>
{% endhighlight %}

### Step 6 Create Magefox\Example\Block\Adminhtml/Hello.php

Normal block class creation. I won't add any methods as I think ypu've got the hang of it by now!

{% highlight php %}
<?php
namespace Magefox\Example\Block\Adminhtml;

class Hello extends \Magento\Backend\Block\Template
{
    // methods here
}
{% endhighlight %}

### Step 7 Create Magefox/Example/view/adminhtml/templates/hello.phtml

Create your template file, add some content and you will see it magically appear within the content of your admin page.

I'm pretty sure you're XML-ed out by now so I'll leave this tutorial here, thanks for reading!