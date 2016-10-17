---
layout: post
title:  "Google Shopping - Outputting Sale Price Effective Date with PHP"
permalink: "/magentofox/google-shopping-outputting-sale-price-effective-date-with-php/"
date:   2016-10-13 20:00:00 +0000
categories: magento google-shopping
author: Adam Moss
comments: true
body_class: magento-fox
---

In a recent project I was building a good ol' fashioned Google Shopping feed, something I've done more times than I've had hot dinners, except this time I encountered a new attribute called _sale_price_effective_date_

After a swift read of [Google's documentation](https://support.google.com/merchants/answer/1196048?hl=en-GB) I realised it wasn't as straightforward as other PHP dates I have outputted in the past. There's a timezone... and a random T right in the middle, and there's a sale FROM and TO date!!

Ultimately, Google wants something like this:

    2011-03-01T16:00-0800/2011-03-03T16:00-0800

As a Magento user I'll show you the fucntion I used to get to this hideous string of data. I've stripped it down slightly for simplicity.

{% highlight php %}
<?php
public function getSalePriceEffectiveDate($product)
    {
        // return if no sale price
        if ($product->getSpecialPrice() <= 0) {
            return false;
        }

        $currentTimeString = strtotime("now");

        $saleFromTimeString = $product->getSpecialFromDate()
            ? strtotime($product->getSpecialFromDate())
            : $currentTimeString;

        $saleToTimeString = $product->getSpecialToDate()
            ? strtotime($product->getSpecialToDate())
            : false;

        if ($saleToTimeString) {

            $from = date("Y-m-d\TH:iO", $saleFromTimeString);
            $to = date("Y-m-d\TH:iO", $saleToTimeString);

            return $from."/".$to;
        }

        return false;
    }
{% endhighlight %}

The key part here is how each date is formatted based on Google's requirements:

    date("Y-m-d\TH:iO", $saleFromTimeString)

The `\T` is what allows the T to intersect the date and time, and the `O` outputs the timezone difference in hours between the server location and GMT.

Another sneaky part of the script is how I've set the `$saleFromTimeString` as the current time in case one hasn't been set. If there's an end date then surely there has to be a start date right?

It's all relatively simple but if you're not familiar with all aspects of the PHP `date()` function it can be tricky! Hopefully this comes in useful for someone.