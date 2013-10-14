---
layout: post
title: "Dynamic routes in Symfony2"
date: 2013-10-01 18:30
comments: true
categories:
    - Symfony
    - router
    - dynamic 
---
Given that most times we need to associate a static route entry with a method of a controller, it is normal for Symfony developers are used to working with the annotation @Route of FrameworkExtraBundle.  

In some cases, it will be interesting or necessary to work with the router to generate dynamic routes. This means that any bundle can generate a route from a service, defining both the name of the route as all the information necessary for the resolution of the route.  

Consider the following example

    <?php

    namespace Mmoreram\AcmeBundle\Router;

    use Symfony\Component\Config\Loader\LoaderInterface;
    use Symfony\Component\Config\Loader\LoaderResolverInterface;
    use Symfony\Component\Routing\Route;
    use Symfony\Component\Routing\RouteCollection;

    /**
     * Acme dynamic router
     */
    class AcmeRoutesLoader implements LoaderInterface
    {
        /**
         * @var boolean
         * 
         * Route is loaded
         */
        private $loaded = false;

        /**
         * Loads a resource.
         *
         * @param mixed  $resource The resource
         * @param string $type     The resource type
         * 
         * @return RouteCollection
         * 
         * @throws RuntimeException Loader is added twice
         */
        public function load($resource, $type = null)
        {
            if ($this->loaded) {

                throw new \RuntimeException('Do not add this loader twice');
            }

            $routes = new RouteCollection();

            /**
             * url('controller_name') will point AcmeController:methodAction()
             */
            $routes->add('controller_name', new Route('controller/route', array(
                '_controller'   =>  'AcmeBundle:Acme:method',
            )));

            $this->loaded = true;

            return $routes;
        }

        /**
         * Returns true if this class supports the given resource.
         *
         * @param mixed  $resource A resource
         * @param string $type     The resource type
         *
         * @return boolean true if this class supports the given resource, false otherwise
         */
        public function supports($resource, $type = null)
        {
            return 'acme' === $type;
        }

        /**
         * Gets the loader resolver.
         *
         * @return LoaderResolverInterface A LoaderResolverInterface instance
         */
        public function getResolver()
        {
        }

        /**
         * Sets the loader resolver.
         *
         * @param LoaderResolverInterface $resolver A LoaderResolverInterface instance
         */
        public function setResolver(LoaderResolverInterface $resolver)
        {
        }
    }

In method `supports()`, `$type` value can be any desired value, and only should be defined once in all project.  

As any service, we must define this class in dependency injection with specific tag.

    services:
        acme.routes.loader:
            class: Mmoreram\AcmeBundle\Router\AcmeRoutesLoader
            tags:
                - { name: routing.loader }

And finally we just need to make our project know where to build our route, so in `routing.yml` file we must add these lines.

    acme_routes:
        resource: .
        type: acme

> At this point, type value must be the same as defined in Router service.
