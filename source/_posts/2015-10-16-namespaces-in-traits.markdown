---
layout: post
title: "Namespaces in Traits"
date: 2015-10-16 12:32
comments: true
author: Marc Morera
categories: 
    - traits
    - namespace
---
Some projects using PHP 5.4 are actually using Traits. If you don't know yet
what a trait is, there are some interesting links for you.

* [https://secure.php.net/manual/en/language.oop5.traits.php](https://secure.php.net/manual/en/language.oop5.traits.php)
* [http://www.sitepoint.com/using-traits-in-php-5-4/](http://www.sitepoint.com/using-traits-in-php-5-4/]
* [http://www.sitepoint.com/php-traits-good-or-bad/](http://www.sitepoint.com/php-traits-good-or-bad/)

This post is about the usage of "use statements" in Traits, like this example

``` php
namespace SomeProject;

use SomeProject\SomeObject;

/**
 * My trait
 */
trait MyTrait
{
    /**
     * Do something
     *
     * @param SomeObject $someObject Some object instance
     */
    public function doSomething(SomeObject $someObject)
    {
        // Do things...
    }
}
```

Is that implementation good enough?

## TL;DR

* Never use "use statements" in Traits. Never.

## The overall picture

When you work on open source, you learn to think about the overall picture. This
means that, probably, your code will somehow, somewhere, part of something even
bigger. Your code must be ready for that.

Let's work with the first shown example. This implementation is correct, right?
I mean, it seems correct because we are not thinking with the global design of
something bigger than our project.

When you work on open source, you learn as well some basic rules. These rules
are based on the reality of sharing. People share their work, and sometimes
means that you will never know what packages will coexist with your own package.

Remember that.

## Friendly packages

Because our package should be able to work with other unknown packages, then we
should design our packages as much isolated as possible. A friendly package is
the one that can live with all type of external packages, even if they are not
friendly at all.

Let me show you why our first example is not friendly at all.
First of all let's see our package again.

``` php
namespace SomeProject;

use SomeProject\SomeObject;

/**
 * My trait
 */
trait MyTrait
{
    /**
     * Do something
     *
     * @param SomeObject $someObject Some object instance
     */
    public function doSomething(SomeObject $someObject)
    {
        // Do things...
    }
}
```

This trait assumes too many things, and because of that, is not friendly at all.
We'll see soon why this implementation can make itself completely non-usable by
many projects.

Let's see another third party project, with another trait, providing us another
useful behavior.

``` php
namespace AnotherProject;

use AnotherProject\SomeObject;

/**
 * My another trait
 */
trait MyAnotherTrait
{
    /**
     * Do another thing
     *
     * @param SomeObject $someObject Some object instance
     */
    public function soAnotherThing(SomeObject $someObject)
    {
        // Do things...
    }
}
```

As you can see, this class belongs to another isolated project, so each one will
work properly in its own sandbox. Until now, everything works perfectly... but
remember when we were talking about the overall picture? In the real world, this
last example is what happens.

``` php
namespace ThirdPartyProject;

use SomeProject\MyTrait;
use AnotherProject\MyAnotherTrait;

/**
 * My super class
 */
class MySuperClass
{
    use MyTrait;
    use MyAnotherTrait;
}
```

What happens here?
Well, as you may know (otherwise, please read about Traits), a Trait is
considered as a simple copy/paste action inside a class. That means that, on the
one hand, all "use statements" are copied one by one on the top of the main 
object, and on the other, all methods, with some priorities considerations, are 
copied inside the main class.

So, our last example equals that new one.

``` php
namespace ThirdPartyProject;

use SomeProject\SomeObject;
use AnotherProject\SomeObject;

/**
 * My super class
 */
class MySuperClass
{
    /**
     * Do something
     *
     * @param SomeObject $someObject Some object instance
     */
    public function doSomething(SomeObject $someObject)
    {
        // Do things...
    }
    
    /**
     * Do another thing
     *
     * @param SomeObject $someObject Some object instance
     */
    public function soAnotherThing(SomeObject $someObject)
    {
        // Do things...
    }
}
```

Oh, wait... can you see that?

``` php
use SomeProject\SomeObject;
use AnotherProject\SomeObject;
```

As you may think, this is il·legal in PHP.

## Complete namespaces

Considering that all traits should be friendly then, the best solution here is
avoiding namespaces. You could tell right now...

```
- Oh, well Marc, you can have this problem as well with methods. What happens if
  two traits have same method definitions?
- Well, when you use a trait, then you should revise these parts that it offers
  you (class signature, methods signature...). You should not take in account if
  that trait uses internal statements or not, considering that as possible 
  incompatibilities with other external traits.
```

So, this is our first example transformed as a friendly trait.

``` php
namespace SomeProject;

/**
 * My trait
 */
trait MyTrait
{
    /**
     * Do something
     *
     * @param \SomeProject\SomeObject\SomeObject $someObject Some object instance
     */
    public function doSomething(\SomeProject\SomeObject\SomeObject $someObject)
    {
        // Do things...
    }
}
```

Same trait, same behavior, but more friendly.