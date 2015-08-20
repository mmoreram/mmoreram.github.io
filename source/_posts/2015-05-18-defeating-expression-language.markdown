---
layout: post
title: "Defeating Expression Language"
date: 2015-05-18 09:53
comments: true
author: Marc Morera
categories: 
    - Symfony
    - Expression Language
    - factory
---
How beautiful Expression Language definitions are, right? I mean, inserting that
complex expressions in a Dependency Injection configuration file is so nice and
fast if you need to inject the result of a method in a service (one of the
multiple examples we can see)

Let's see a simple example of how we use this library.

``` yaml
services:
    
    #
    # My managers
    #
    some_manager:
        class: This\Is\My\Manager
        arguments:
            - @=service("another_manager").someCall("value")
            - @=service("yet_another_manager").getInjectableInstance(parameter("my_parameter"))
```

This is not a bad idea, really, but because we are engineers and we should have
as much information as possible in order to be able to choose between the best
option, always, I will show you another way of defining this piece of code.

Let's do that!

### Factories

Remember the [Factory pattern in Symfony2](http://mmoreram.com/blog/2013/12/23/factory-pattern-in-symfony2/) 
post I wrote some time ago? I talked about how this pattern can be implemented 
in your Symfony projects.

Well, just for your information, most of your Expression Language definitions 
can be nicely done using Factories.

Let's reproduce the same example using factories.

``` yaml
services:
    
    #
    # My managers
    #
    some_manager:
        class: This\Is\My\Manager
        arguments:
            - @my_injectable_value
            - @my_injectable_service
            
    my_injectable_value:
        class: StdClass
        factory: 
            - @another_manager
            - someCall
        arguments:
            - value
            
    my_injectable_service:
        class: This\Is\My\Injectable\Class
        factory: 
            - @yet_another_manager
            - getInjectableInstance
        arguments:
            - %my_parameter%
```

### Dependency

Take a look and realise that we've removed a package dependency from your 
project, as you don't need `symfony/expression-language` anymore, and your DIC
definition will be easily exportable to another format if someday is needed.

### Reflexion

Is Expression Language a bad choice? Well, only you should be able to know if
using this library is a good choice or not, because only you know your needs and
your resources, but every time you add a new Expression Language line, just ask
yourself...

Can I use simple DI definitions here? Is the only way I can do that?