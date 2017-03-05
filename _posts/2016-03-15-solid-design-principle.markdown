---
layout: post
title:  "Simple PHP Guide to S.O.L.I.D Design Principles"
permalink: "/magentofox/simple-php-guide-to-solid-design-principles/"
date:   2016-03-15 10:00:00 +0000
categories: magento
author: Adam Moss
comments: true
body_class: magento-fox
reading-time: 15 mins
---

**S.O.L.I.D** is a coding standard that all PHP developers should be familiar with when undertaking Magento module development. It was introduced over a decade ago by Robert C Martin (Uncle Bob) and is used across the object-oriented design spectrum. Why use these principles I hear you ask? When applied properly it makes your code more extendable, logical and easier to read.

I'm going to try and keep this as jargon-free as possible so it's easy for beginners to understand. Let's go through each principle one by one:

### S: Single Responsibility Principle

Each class should have *one* responsibility and *one* responsibility only. This means that all the methods and properties should all work towards the same goal.

Lets take the following simple class as an example of how this principle may be violated:

{% highlight php %}
<?php 
class Customer
{
    public $name;
    public $age;

    public function __contsruct($name, $age)
    {
        $this->name = $name;
        $this->age = $age;
    }

    public function getCustomerDetails()
    {
        return "Name: ".$this->name." Age: ".$this->age;
    }
}
{% endhighlight %}

This may be fine for printing information to the frontend but what about if you need to get that information in JSON or as an array? You use classes or interfaces with defined roles, thus separating the business logic from the presentation layer:

{% highlight php %}
<?php 
interface CustomerInterface
{
    function getCustomerDetails();
}
{% endhighlight %}

We then may have an interface for retail customers that must return the data as a string.

{% highlight php %}
<?php 
class RetailCustomer implements CustomerInterface
{
    public function getCustomerDetails()
    {
        return "Name: ".$this->name." Age: ".$this->age;
    }
}
{% endhighlight %}

We may have the trade interface to return this data as an array:

{% highlight php %}
<?php 
class TradeCustomer implements CustomerInterface
{
    public function getCustomerDetails()
    {
        return array( "name" => $this->name, "age" => $this->age);
    }
}
{% endhighlight %}

### O: Open-closed Principle

In a nutshell your classes should be extendable without actually changing the contents of the class you're extending. Here's a very basic example of how this principle can be violated if we had to introduce another customer type:

{% highlight php %}
<?php 
class PointsCalculator
{
    public function getPointsSpent($customerType, $points)
    {
        switch ($customerType)
        {
            case 'retail':
            return $points * 0.5;
            break;

            case 'trade':
            return $points * 0.75;
            break;
        }
    }
}
{% endhighlight %}

The way around this would be to use interfaces as in the previous principle:

{% highlight php %}
<?php 
interface CustomerInterface
{
    function getPointsSpent();
}

class RetailCustomer implements CustomerInterface
{
    public function getPointsSpent()
    {
        return $this->type * 0.5;
    }
}

class TradeCustomer implements CustomerInterface
{
    public function getPointsSpent()
    {
        return $this->type * 1;
    }
}

class PointsCalculator
{
    public function getPointsSpent($customerTypes)
    {
        $amount = 0;

        foreach ($customerTypes as $type)
        {
            $amount += $type->getPointsSpent();
        }

        return $amount;
    }
}

{% endhighlight %}

### L: Liskov Substitution Principle

This principle basically specifies that child classes should be suitable for their parent classes. This means it will adhere to the principles and functionality of the class it extends.

{% highlight php %}
<?php 
class Instrument 
{
    public function play() {..}
    public function changeKey() {..}
    public function autoTune() {..}
    public function plugIn() {..}
}
{% endhighlight %}

The instrument class may behave as an abstract and could be implemented in the following ways:

{% highlight php %}
<?php 
class Guitar extends Instrument
{
    public function play() {
        $this->plugIn();
        $this->strum();
        parent::play();
    }

    public function strum() {..}
}

class Trumpet extends Instrument
{
    public function play() {
        $this->blow();
        parent::play();
    }

    public function blow() {..}
}
{% endhighlight %}

### I: Interface Segregation Principle

This principle is probably the simplest in terms of theory. A client should not be forced to use interfaces that it doesn't need. Here's an example of violation of this principle:

{% highlight php %}
<?php 
interface InstrumentInterface {
    public function play();
    public function changeKey();
    public function autoTune();
    public function plugIn();
}

class Guitar implements InstrumentInterface {
    public function play() {..}
    public function changeKey() {..}
    public function autoTune() {..}
    public function plugIn() {..}
}

class Trumpet implements InstrumentInterface {
    public function play() {..}
    public function changeKey() {..}
    public function autoTune() { /* Exception */ }
    public function plugIn() { /* Exception */ }
}
{% endhighlight %}

The Trumpet class has autoTune() and plugIn() forced upon it - I'm not saying that there's no plugin-in trumpets out there but in this case I'm talking about a standard acoustic trumpet. An ISP-safe method is to create an interface for each of the instruments that employs only the methods needed for the client:

{% highlight php %}
<?php 
interface InstrumentInterface 
{
    public function play();
    public function changeKey();
}

interface GuitarInterface 
{
    public function autoTune();
    public function plugIn();
}

class Guitar implements InstrumentInterface, GuitarInterface
{
    public function play() {..}
    public function changeKey() {..}
    public function autoTune() {..}
    public function plugIn() {..}
}

class Trumpet implements InstrumentInterface {
    public function play() {..}
    public function changeKey() {..}
}
{% endhighlight %}

### D: Dependency Inversion Principle

This principle states that high level modules should not depend on low level modules. High level modules should never change and should be decoupled (seperated) from low level modules that could be changed on a daily basis.

The introduction of dependency injection in Magento 2 has made this principle easier to adhere to, as it is now clear which classes and clients are dependent on each other.

Take the example below which shows a rather limited example of higher level code being dependent on lower level code:

{% highlight php %}
<?php 
class CountApples
{
    private $apple;
    public function __contsruct($apple)
    {
        $this->apple = $apple;
    }

    public function howMany()
    {
        return count($this->apple->howMany());
    }
}
{% endhighlight %}

This does the job, but the class is completely dependent on us always counting apples. There are other fruits out there kids! We should make our counting class more general:

{% highlight php %}
<?php 
class CountFruit
{
    private $fruit;
    public function __contsruct($fruit)
    {
        $this->fruit = $fruit;
    }

    public function howMany()
    {
        return count($this->fruit->howMany());
    }
}

/** Create our generic fruit interface **/

interface Fruit
{
    public function howMany();
}

/** Create our clients **/

class Apples implements Fruit 
{
    public function howMany()
    {
        return 14;
    }
}

class Bananas implements Fruit 
{
    public function howMany()
    {
        return 6;
    }
}
{% endhighlight %}

The end result is code that is less dependent on the lower level code with which it is working. You could take this much further than the example above but the theory is the same.

I hope I have kept this simple enough for beginner OOPHP developers to understand. Thanks for reading.