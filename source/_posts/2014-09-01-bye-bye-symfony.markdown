---
layout: post
title: "Bye Bye Symfony"
date: 2014-09-01
comments: true
author: Marc Morera
categories:
    - Symfony
    - ports
    - adapters
---
Don't be afraid. You know what I think about the Symfony components and how I
love them. Until now any circumstance has changed my mind about using them in my
projects, and over the time, after discover every hidden single piece of code, I
have turned more amazed of their power.

The reason of this post is just to tell you, with a simple example, how to say
*Bye Bye, Symfony!* and say *Hi PHP!*. This really means uncouple from Symfony
Components and still use them as the default implementation, while we can
securely remove, from the composer *require* block, our Symfony dependencies.

One of the poor things I have heard these last months has been the word *you must*
or *you should* event without knowing exactly what is the real direction of my
project, so every word in this post will be just an idea you could have in mind
when you design your own architecture, thinking about your real needs, far from
the things you should or you must do.

To explain that, how we can say goodbye to Symfony, we will start with a simple
example. A simple class we could find in every project. Our class will have no
sense but will be useful in our specific example, helping us to understand what
is all this about.

``` php
<?php

/**
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 *
 * Feel free to edit as you please, and have fun.
 *
 * @author Marc Morera <yuhu@mmoreram.com>
 */

namespace Package\Generator;

use Symfony\Component\Routing\Generator\UrlGeneratorInterface;

/**
 * Class UrlPrinter
 */
class UrlPrinter
{
    /**
     * @var UrlGeneratorInterface
     *
     * Url generator
     */
    protected $urlGenerator;

    /**
     * Construct method
     *
     * @param UrlGeneratorInterface $urlGenerator Url generator
     */
    public function __construct(UrlGeneratorInterface $urlGenerator)
    {
        $this->urlGenerator = $urlGenerator;
    }

    /**
     * Generates Homepage route and print complete path
     *
     * @return string Route path
     */
    public function printHomePageRoute()
    {
        return $this
            ->urlGenerator
            ->generate('homepage');
    }
}
```

As we can see, use this class means use `UrlGeneratorInterface`. This class is
part of the Symfony Routing Component, so is a really bad idea if you really
want to make your project multi-environment

Is that good? Well, it depends on your both product requirements and technical
requirements. If you want to use always the Symfony router component, this piece
of code is good enough. Otherwise, If you want to use another router, you have a
problem.

Say your project must work with and without Symfony. Our class should not depend
on this interface. One of the strategies to uncouple external implementations is
using ports and adapters.

I will explain how it works and how we can implement it.

Real life example
-----------------

Let's imagine we want to sell an USB device. We have decided in our product
definition that this device will connect to external devices using USB Standard
A (the big one). One of the possible approaches would be to design our device
with the external connector coupled. This means selling it with the USB wire
(like an USB camera, for example).

But why coupling our device with an specific output? Do you know what it really
means?

* If the wire breaks, we must change all the device.
* You can connect with devices that accept only USB-A
* Wire improvements means new device versions.

What would be an alternative decision? Well, why don't we focus our efforts in
the camera and we just let other companies to help us with the wire? This means
more flexibility for the final user and capability of removing and changing the
cable when is broken. And... they could connect their camera to many different
formats, even for the most adventurous, their own formats.

So, in fact, our device should have one female USB port with an specification
of how connect it with another device. The wire would be just an adapter of our
device so many of them could be plugged-in.

We have talked about devices, ports, adapters, specifications, but... how can
this help me with my project? Is that hard?

The answer is No.

Solution
--------

Our goal is to create an specification for URL generators and force each adapter
to implement it in their own way. As we know, the OOP languages resolve the
specification with the interfaces.

Let's create our specification.

``` php
<?php

/**
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 *
 * Feel free to edit as you please, and have fun.
 *
 * @author Marc Morera <yuhu@mmoreram.com>
 */

namespace Package\Specification;

/**
 * Interface PackageUrlGeneratorInterface
 */
interface PackageUrlGeneratorInterface
{
    /**
     * Generates a route given its name.
     *
     * @return string Route path
     */
    public function generateUrl($routeName);
}
```
We are just specifying. We want that, whatever is the implementation, each
adapter must fulfill this rule: Has a method called `generateRoute` that, given
the route name, will return its complete path.

Let's adapt our UrlPrinter to work with specification instead of implementation.

``` php
<?php

/**
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 *
 * Feel free to edit as you please, and have fun.
 *
 * @author Marc Morera <yuhu@mmoreram.com>
 */

namespace Package\Generator;

use Package\Specification\PackageUrlGeneratorInterface;

/**
 * Class UrlPrinter
 */
class UrlPrinter
{
    /**
     * @var PackageUrlGeneratorInterface
     *
     * Url generator
     */
    protected $urlGenerator;

    /**
     * Construct method
     *
     * @param UrlGeneratorInterface $urlGenerator Url generator
     */
    public function __construct(PackageUrlGeneratorInterface $urlGenerator)
    {
        $this->urlGenerator = $urlGenerator;
    }

    /**
     * Generates Homepage route and print complete path
     *
     * @return string Route path
     */
    public function printHomePageRoute()
    {
        return $this
            ->urlGenerator
            ->generateUrl('homepage');
    }
}
```

Now, our UrlPrinter does not depends on any external library. Goal reached!

But, now what? I mean, we should implement our adapter, am I right? What is a
camera with a great specification and without wire plugged in? (A wireless
camera, welcome to the twenty century ^^. Just joking)

Our UrlPrinter needs a *PackageUrlGeneratorInterface* implementation to be
built, so we have to implement an adapter. And because we still want to work with
Symfony Routing Component as my first option, we will create the Symfony Adapter
for my specification.

``` php
<?php

/**
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 *
 * Feel free to edit as you please, and have fun.
 *
 * @author Marc Morera <yuhu@mmoreram.com>
 */

namespace Package\Adapter;

use Package\Specification\PackageUrlGeneratorInterface;
use Symfony\Component\Routing\Generator\UrlGeneratorInterface;

/**
 * Class SymfonyUrlGeneratorAdapter
 */
class SymfonyUrlGeneratorAdapter implements PackageUrlGeneratorInterface
{
    /**
     * @var UrlGeneratorInterface
     *
     * Url generator
     */
    protected $urlGenerator;

    /**
     * Construct method
     *
     * @param UrlGeneratorInterface $urlGenerator Url generator
     */
    public function __construct(UrlGeneratorInterface $urlGenerator)
    {
        $this->urlGenerator = $urlGenerator;
    }

    /**
     * Generates a route given its name
     *
     * @return string Route path
     */
    public function generateUrl($routeName)
    {
        $this
            ->urlGenerator
            ->generate($routeName);
    }
}
```

This an adapter, my friends. Each adapter has one mission, and is just transform
the way we understand the method `generateUrl` must work (specification) to the
way each external project understands it (implementation). In this case, very
easy, our `generateUrl` is the same as Symfony Router Component's `generate`.

Symfony Component is now required by this adapter, but because we are not
required to use this adapter (Maybe we can use a dummy one for our tests, or a
mocked one), Symfony Component is not required by our package anymore, just
suggested.

Let's see another Adapter implementation, requiring another external library.

``` php
<?php

/**
 * For the full copyright and license information, please view the LICENSE
 * file that was distributed with this source code.
 *
 * Feel free to edit as you please, and have fun.
 *
 * @author Marc Morera <yuhu@mmoreram.com>
 */

namespace Package\Adapter;

use Package\Specification\PackageUrlGeneratorInterface;
use AnotherProject\UrlGeneratorInterface;

/**
 * Class SymfonyUrlGeneratorAdapter
 */
class SymfonyUrlGeneratorAdapter implements PackageUrlGeneratorInterface
{
    /**
     * @var UrlGeneratorInterface
     *
     * Url generator
     */
    protected $urlGenerator;

    /**
     * Construct method
     *
     * @param UrlGeneratorInterface $urlGenerator Url generator
     */
    public function __construct(UrlGeneratorInterface $urlGenerator)
    {
        $this->urlGenerator = $urlGenerator;
    }

    /**
     * Generates a route given its name
     *
     * @return string Route path
     */
    public function generateUrl($routeName)
    {
        $routeName = '_' . $routeName;

        $this
            ->urlGenerator
            ->anotherGenerateMethod($routeName);
    }
}
```

Both adapters would be placed in the `Adapter/` folder, and the final user
should be able to switch them and even implement new ones.

From require to suggest
--------------------

With our changes, we can remove "symfony/routing" from the require block and add
it into the suggest block in `composer.json` file.

``` yml
"require": {
    "php": ">=5.4",
    ...
},
"suggest": {
    "symfony/routing": "Required if using Routing adapter",
    "mmoreram/another-project": Required if using AnotherProject Routing adapter"
},
```

We will only require the packages needed by the adapter we are using.

What we win
-----------

Much. We win maximum implementation flexibility and minimum coupling. Would be
wise to say that a PHP project should tend to this thought, but once again, it
depends on many factors.

What we lose
------------

It Depends. If you want your project to be understandable by a lot of developing
knowledge levels, this architecture goes away from the comprehensibility of a
simple code. You also can lose by having lot of files, so you should respect
some kind of best practices code, to make people working on your project
comfortable dealing with it.

Best practices
--------------

I should talk about *Marc practices* instead of *Best practices*. I am used to
adding my adapters always in the folder `/Adapter/{PortName}/`, being *PortName*
the name of the port we are dealing with in camel case format. In this case, we
should add both adapters in `/Adapter/UrlGenerator/`.

Given this format we can determine what adapters we can use in our project in a
very agile way.

Reference projects
------------------

First project that comes in my mind when I think about Ports/Adapters is
[Gaufrette](https://github.com/KnpLabs/Gaufrette/tree/master/src/Gaufrette), a
filesystem abstraction layer with a lot of adapters implemented.

In [Elcodi](https://github.com/elcodi/elcodi) we are actually using this
library and its really awesome how easy is using it.

We have also implemented internally some features using this architecture, for
example, the [Geo Schema Populator](https://github.com/elcodi/elcodi/tree/master/src/Elcodi/Component/Geo/Adapter/Populator),
a new feature under development that will allow you to populate all Geo schema
using some adapters. Right now only GeoData adapter is implemented.

Another example, the
[Currency Exchange Rates Populator](https://github.com/elcodi/elcodi/tree/master/src/Elcodi/Component/Currency/Adapter/CurrencyExchangeRatesProvider)
with OpenExchangeRates implementation already done.

Final thoughts
--------------

Using ports and adapters is really a great tool for those who want to uncouple
from implementations and a great pattern if you develop open source. Open source
should satisfy as people as possible, so remember, specify and then implement.

Try it and then tell us your experience :)