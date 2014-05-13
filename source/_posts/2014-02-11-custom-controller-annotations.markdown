---
layout: post
title: "Custom controller annotations"
date: 2014-02-11 20:21
comments: true
categories: 
- Annotations
- Symfony
---

The goal is easy: To provide a very intuitive and easy way of creating
controller annotations in Symfony2.  

[ControllerExtraBundle](https://github.com/mmoreram/ControllerExtraBundle)
is both a set of annotations and a platform for creating new custom ones.
Actually, the bundle can create a new Entity, like ParamConverter does, with in
this case is just an empty Entity. The bundle also can create FormTypes, Forms
and FormViews with several options. Can also automate the doctrine `flush()`
action after the method execution or log something before / after every desired 
action.

Bundle is still being tested in production environments, all feedback will be
very very useful.

Let's see how can we create our Annotations.

* Annotation
* Resolver
* Definition
* Registration

## Annotation

The annotation object. You need to define the fields your custom annotation
will contain.
Must extends `Mmoreram\ControllerExtraBundle\Annotation\Abstracts\Annotation`
abstract class.

``` php
<?php

namespace My\Bundle\Annotation;

use Mmoreram\ControllerExtraBundle\Annotation\Abstracts\Annotation;

/**
 * Entity annotation driver
 *
 * @Annotation
 */
class MyCustomAnnotation extends Annotation
{

    /**
     * @var string
     *
     * Dummy field
     */
    private $field;
    
    
    /**
     * Get Dummy field
     *
     * @return string Dummy field
     */
    public function getField()
    {
        return $this->field;
    }
}
```

## Resolver

Once you have defined your own annotation, you have to resolve how this
annotation works in a controller. You can manage this using a Resolver. Must
extend `Mmoreram\ControllerExtraBundle\Resolver\Interfaces\AnnotationResolverInterface;`
abstract class.

``` php
<?php

namespace My\Bundle\Resolver;

use Symfony\Component\HttpFoundation\Request;
use Mmoreram\ControllerExtraBundle\Resolver\Interfaces\AnnotationResolverInterface;
use Mmoreram\ControllerExtraBundle\Annotation\Abstracts\Annotation;

/**
 * MyCustomAnnotation Resolver
 */
class MyCustomAnnotationResolver implements AnnotationResolverInterface
{

    /**
     * Specific annotation evaluation.
     * This method MUST be implemented because is defined in the interface
     *
     * @param Request          $request    Request
     * @param Annotation       $annotation Annotation
     * @param ReflectionMethod $method     Method
     *
     * @return MyCustomAnnotationResolver self Object
     */
    public function evaluateAnnotation(
                                        Request $request, 
                                        Annotation $annotation, 
                                        ReflectionMethod $method )
    {
        /**
         * You can now manage your annotation.
         * You can acced to its fields using public methods.
         * 
         * Annotation fields can be public and can be acceded directly,
         * but is better for testing to use getters; they can be mocked.
         */
        $field = $annotation->getField();
        
        /**
         * You can also access to existing method parameters.
         * 
         * Available parameters are:
         * 
         * # ParamConverter parameters ( See `resolver_priority` config value )
         * # All method defined parameters, included Request object if is set.
         */
        $entity = $request->attributes->get('entity');
        
        /**
         * And you can now place new elements in the controller action.
         * In this example we are creating new method parameter
         * called $myNewField with some value
         */
        $request->attributes->set(
            'myNewField',
            new $field()
        );
        
        return $this;
    }

}
```

This class will be defined as a service, so this method is computed just
before executing current controller. You can also subscribe to some kernel
events and do whatever you need to do ( You can check
`Mmoreram\ControllerExtraBundle\Resolver\LogAnnotationResolver` for some
examples.

## Definition

Once Resolver is done, we need to define our service as an Annotation
Resolver. We will use a custom `tag`.

``` yml
parameters:
    #
    # Resolvers
    #
    my.bundle.resolvers.my_custom_annotation_resolver.class: My\Bundle\Resolver\MyCustomAnnotationResolver

services:
    #
    # Resolvers
    #
    my.bundle.resolvers.my_custom_annotation_resolver:
        class: %my.bundle.resolvers.my_custom_annotation_resolver.class%
        tags:
            - { name: controller_extra.annotation }
```

## Registration

We need to register our annotation inside our application. We can just do it in
the `boot()` method of `bundle.php` file.

``` php
<?php

namespace My\Bundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;
use Doctrine\Common\Annotations\AnnotationRegistry;

/**
 * MyBundle
 */
class ControllerExtraBundle extends Bundle
{

    /**
     * Boots the Bundle.
     */
    public function boot()
    {
        $kernel = $this->container->get('kernel');

        AnnotationRegistry::registerFile($kernel
            ->locateResource("@MyBundle/Annotation/MyCustomAnnotation.php")
        );
    }
}
```

*Et voilà!*  We can now use our custom Annotation in our project controllers.