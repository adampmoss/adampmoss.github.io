---
layout: post
title:  "Magento 1 - Extending Core JavaScript with Prototype"
permalink: "/magentofox/magento1-extending-core-javascript-with-prototype/"
date:   2016-04-20 20:00:00 +0000
categories: magento
author: Adam Moss
comments: true
---

The Majority of Magento 1's JavaScript is written in the very misunderstood DOM-extending Prototype framework. Launched in 2005, Prototype has been around for a long-ass time and is struggling to mintain relevance in the face of modern, package-based frameworks like React.js.

For Magento developers however it still plays a big part of every day life and when customising Magento there is often the need to interact with the core JavaScript files. When extending skin JS the temptation is to put a modified copy of the file into your theme, but in the instance of an upgrade this will heavily impact your workload.

Luckily, Prototype makes it easy to change the output and input of class methods using two methods of its own:

- [Class#addMethods](http://api.prototypejs.org/language/Class/prototype/addMethods/)
- [Function#wrap](http://prototypejs.org/doc/latest/language/Function/prototype/wrap/)

I'm gonna go through how to put this into practice using a default Magento 1 core Prototype class.

### addMethods

addMethods allows you to take a class that has already been created and initialised and (as you'd expect) add new methods, which can then be accessed by the object instance. The method takes place on the class definition. It also allows us to overwrite existing methods simply by defining them again. Let's take a look at an example:

Let's say we want to add 'plus tax' to the price presented by a configurable product. Well, in the `Product.Config` class found in **/js/varien/configurable.js** the method that formats the price before it is presented is (unsurprisingly) `formatPrice`. I'm going to use `addMethods` to change how this is output.

The first step is to create a module that will include a custom JS file on the product page - I'm going to assume you know how to do this bit. The name of my file is shown below:

#### skin/frontend/base/default/js/magentofox/custom.js

{% highlight js %}
Product.Config.addMethods({

    formatPrice: function(price, showSign){
        var str = '';
        price = parseFloat(price);
        if(showSign){
            if(price<0){
                str+= '-';
                price = -price;
            }
            else{
                str+= '+';
            }
        }

        var roundedPrice = (Math.round(price*100)/100).toString();

        if (this.prices && this.prices[roundedPrice]) {
            str+= this.prices[roundedPrice];
        }
        else {
            str+= this.priceTemplate.evaluate({price:price.toFixed(2)});
        }
        return str + Translator.translate(" plus tax");
    }
});
{% endhighlight %}

All I've done here is replicated the method in its entirety and customised it where necessary. If I didn't replicate the method then the functionality in the original method would have been lost. In fact the only line I've changed is this part: `return str + Translator.translate(" plus tax");`. 

This achieves the end goal without having to edit or duplicate the whole of the original file. Easy!

### wrap

Wrap is kinda similar to addMethods but also very different. Rather than taking place on the class definition, wrap targets the method themeselves. Rather like [plugins in Magento 2](/magentofox/magento-2-basics-part-6-using-plugins/), wrap allows you to modify the arguments before a method is called, completely rewrite a method and change the output of any given method.

Let's take the last example again, but this time we want to change the value of the price going into the method. Yeah, you'd never need to do this but it demonstrates the point:

{% highlight js %}
Product.Config.prototype.formatPrice = Product.Config.prototype.formatPrice.wrap(
    function(originaMethod, price, showSign) {
        price = 600;
        var str = originaMethod(price, showSign);

        return str + Translator.translate(" plus tax");
    });
{% endhighlight %}

So what's going on above? Well once we're inside the wrap function there's a new function declared: `function(originaMethod, price, showSign)`. The first argument is always the method itself, and the other arguments are arguments that should get passed to the method. That means that as soon as we are inside this function we have access to those arguments.

So on the next line I've decided to change the price so that it's always 600. Now I can pass my changed argument into the method in question which happens on the next line `var str = originaMethod(price, showSign);`. The method will now run as normal and return whatever it returns - this gets saved into my newly created `str` variable.

Finally I'm going to return the function properly (after it has now successfully ran with all the functionality needed) and I'm going to add my text to the end.

This example is rather extreme in that modifies the before and after parts of the `formatPrice` method.

### Summary

There is also another method called [Object.extend](http://api.prototypejs.org/language/Object/extend/) that allows core classes and methods to be extended without touching the core, but I have left this out of this article because it involves create new class definitions which is slightly different. Using this would mean having to change the class instances across Magento which could be more time consuming.

Hope you enjoyed the post, feel free to leave comments.