---
layout: post
title: "Factory pattern in Symfony2"
date: 2013-12-23 12:10
comments: true
categories:
- Symfony2
- Factory
- Pattern
- Finder
---

## The pattern.

The factory pattern gives responsibility to create some instances of certain
types to a class called factory. All possible classes that can instantiate the
factory should always implement an interface, so we will be able to call certain
methods, whether the class is. Let's see an specific and usable example.

First of all, we have an interface called LoggerInterface. Is just an interface
so just define all methods that implementations must implement.

``` php
<?php

namespace My\Bundle\Namespace;

/**
 * Interface for all loggers
 */
interface LoggerInterface
{
    /**
     * Logs message given as parameter
     *
     * @param string $message Message to log
     *
     * @return LoggerInterface self Object
     */
    public function log($message);
}
```

On the one hand we have an implementation called FileLogger. This class just
write given messages in a file.

``` php
</php

namespace My\Bundle\Namespace;

use My\Bundle\Namespace\LoggerInterface;

/**
 * This class logs into specific file
 */
class FileLogger implements LoggerInterface
{

   /**
    * @var file path
    *
    * File path
    */
    private $filepath = '/tmp/myfile.log';


    /**
     * Logs message given as parameter
     *
     * @param string $message Message to log
     *
     * @return FileLogger self Object
     */
    public function log($message)
    {
        $this->file_put_contents($this->filepath, $message);

        return $this;
    }
}
```

And on the other hand we have a ScreenLogger, that just echoes message as it
comes.

``` php
</php

namespace My\Bundle\Namespace;

use My\Bundle\Namespace\LoggerInterface;

/**
 * This class logs into specific file
 */
class ScreenLogger implements LoggerInterface
{

    /**
     * Logs message given as parameter
     *
     * @param string $message Message to log
     *
     * @return FileLogger self Object
     */
    public function log($message)
    {
        echo $message;

        return $this;
    }
}
```

Given a project we may want to specify using configurarion which Logger we want
to use. Since we want to work with dependency injection component offered by
Symfony2 framework, and any class where we will use our Logger is responsible
for knowing as each instance of our Loggers, we need to create a Factory that
is responsible for taking the setting you have specified, return us an
instance of the class we want.

Given this configuration

``` yml
# /app/config/config.yml

logger:
    type: screen
```

We define our factory service. As `type` configuration value is a free text
value, if value do not references any Logger type we will throw an exception.

``` php
<?php

namespace My\Bundle\Namespace;

/**
 * This class is just a Logger factory
 */
class LoggerFactory
{
    /**
     * Create staticly desired Logger
     *
     * @param string $type Type of Logger to create
     *
     * @return LoggerInterface Logger instance
     */
    static public function get($type)
    {
        $instance = null;

        switch ($type) {

            case 'screen':
                $instance = new ScreenLogger();
                break;

            case 'file':
                $instance = new FileLogger();
                break;

            default:
                throw new BadLoggerDefinitionException;
        }

        return $instance;
    }
}
```

And Factory dependency injection definition.. As factory must not be instanced
to create a Logger ( `get` method is static ) we must use `factory_class` to
define the Factory namespace.

``` yml
# /my/bundle/Namespace/Resources/config/services.yml

services:
    my.logger:
        class: My\Bundle\Namespace\LoggerInterface
        factory_class: My\Bundle\Namespace\LoggerFactory
        factory_method: get
        arguments:
            logger_type: %logger.type%

    my.service:
        class: My\Bundle\Namespace\MyService
        arguments:
            my_logger: @my.logger
```

## Prototyping

Let's take a look at Symfony2 component
[Finder](http://symfony.com/doc/current/components/finder.html) class. We have a
class named Manager that uses this class.

``` php
<?php

namespace My\Bundle\Namespace;

use Symfony\Component\Finder\Finder;

/**
 * This class is just a manager
 */
class Manager
{

    /**
     * Do something
     */
    public function doSomething()
    {
        $finder = new Finder();
        $finder
            ->files()
            ->in(__DIR__);
    }
}
```

Placing this `new Finder` inside Manager class, we assume that Manager has
responsability to know how `Finder` must be built. This creates dependency
between both objects, and that's wrong. So the point is that we should inject a
new instance of `Finder` each time we call doSomething.


``` php
<?php

namespace My\Bundle\Namespace;

use Symfony\Component\Finder\Finder;

/**
 * This class is just a manager
 */
class Manager
{

    /**
     * @var Finder
     *
     * Finder
     */
    private $finder;


    /**
     * Construct method
     *
     * @param Finder $finder Finder
     */
    public function __construct(Finder $finder)
    {
        $this->finder = $finder;
    }


    /**
     * Do something
     */
    public function doSomething()
    {
        $this
            ->finder
            ->files()
            ->in(__DIR__);
    }
}
```

And how do we resolve this problem using Dependency Injection? Is as easy as
creating a new service using Finder as class, with prototype scope.

``` yml
# /my/bundle/Namespace/Resources/config/services.yml

services:
    my.finder:
        class: Symfony\Component\Finder\Finder
        scope: prototype

    my.manager:
        class: My\Bundle\Namespace\Manager
        arguments:
            my_finder: @my.finder
```

If we tale a look at `Finder` class we shall notice that have a static factory
method inside. If we use Factory pattern using this method, we will instanciate
a new `Finder` object the first time, but not the others.
