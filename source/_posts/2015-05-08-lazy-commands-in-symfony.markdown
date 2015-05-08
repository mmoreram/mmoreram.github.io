---
layout: post
title: "Lazy Commands in Symfony"
date: 2015-05-08 11:04
comments: true
categories: 
    - Symfony
    - Commands
    - lazy
---
Have you ever had this scenario?

``` bash
$ php app/console doctrine:database:drop --force
$ php app/console doctrine:database:create

    [Doctrine\DBAL\Exception\ConnectionException]                                     
    An exception occured in driver: SQLSTATE[42000] [1049] Unknown database 'mydatabase'
```

Well, it happens and I will tell you why.

## Command as a Service

Since Symfony version 2.4 you can define your controllers and commands as 
services. This is so useful as long as you need to treat your classes as much
decoupled as possible. You can check some information about how to define them 
as services in 
[Symfony Documentation](http://symfony.com/doc/current/cookbook/console/commands_as_services.html).

Then, let's figure out that our command is intended to check an entity from your
database. Of course, your command should be as empty as possible, placing all 
your business logic inside your service layer (this is not the only strategy, of
course, but there is no strategy where you lace your logic inside your command).

Then, using commands as services, you will have this

* ObjectManager as a service (or Repository)
* Your service, intended to do whatever you need to do, for example, check that
your entities are all enabled. Your ObjectManager or Repository will be injected
here
* Your command, intended to work as the simple layer between your cli interface
and your service layer. Your service will be injected here.

Given this schema, when we require this command through the DI Container, of 
course a new Service instance will be created in order to inject it through the
command constructor, and the object will have to be created as well to be 
injected inside the service.

It means create a new connection to the database. Fair enough till now :)

## Commands list

Let me show you some lines of code. This method is placed in a class called
`Application` inside the bundle `FrameworkBundle`. This class is intended to add
the possibility of adding Commands as services in the main Application class of
Command Component.

[Symfony\Bundle\FrameworkBundle\Console\Application](https://github.com/symfony/symfony/blob/2.7/src/Symfony/Bundle/FrameworkBundle/Console/Application.php)

``` php
protected function registerCommands()
{
    $container = $this->kernel->getContainer();
    foreach ($this->kernel->getBundles() as $bundle) {
        if ($bundle instanceof Bundle) {
            $bundle->registerCommands($this);
        }
    }
    if ($container->hasParameter('console.command.ids')) {
        foreach ($container->getParameter('console.command.ids') as $id) {
            $this->add($container->get($id));
        }
    }
}
```

Oh, wait... what?

Register a command means instantiate it! So if we have a command with a service
injected which has an object manager injected... then we have a problem. If we
don't have the database created we will not be able to create it using the 
Doctrine command.

*~ironic~ Perfect scenario for deployment ~/ironic~*

## Using Lazy services

Of course we should find a solution for this scenario, in order to be able to
call this command when is needed.

When we define as lazy a service, this is not instanced when is injected, but 
only when is accessed. You can find some information about lazy services in
[Symfony Documentation](http://symfony.com/doc/current/components/dependency_injection/lazy_services.html).

The point here is to define our service intended to work with the model as lazy.
The result will be that when we instance the Command, then a proxy object is
created and injected with all the service information. 

Because our command will not be used as long as we don't need it, then the 
service will not be instanced and the ObjectManager not created. We will be able 
to list all the commands, and finally, call
`php app/console doctrine:database:create` properly.

## Implementation

Some tips here...

* Why instancing all services when we just need them to be listed? Is it really
necessary? Doing than we are forcing some service to be defined as lazy just 
because of it, and this is not and will never be a good practice.
* If the command needs to be instanced to be listed, and assuming that this 
information ~~should~~ could be cached, then, is it necessary to call the 
constructor? We could get the class, build the object using `\ReflectionClass`
and request all the needed information.

I invoke the community for some feedback on that. If this is really a need for
some people, we should do some push (and organize us for an implementation, 
maybe) to change this implementation for next Symfony 3 version.

Feedback, feedback :)