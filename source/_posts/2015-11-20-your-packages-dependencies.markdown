---
layout: post
title: "Your packages dependencies"
date: 2015-11-20 21:50
comments: true
author: Marc Morera
categories: 
    - component
    - bundle
    - composer
    - dependencies
    - symfony
---
* A component should add as dependency all needed packages to have a complete
  functionality
* A bundle should add as dependency as well all these bundles that define other 
  services used in your own service definitions
* Take care of your version policy. Being too restrictive reduces the
  compatibility with other packages.

## Responsibilities

- Hi. I'm Marc and I am an open source addict
- Hi Marc, welcome!

That is a reality in fact. I'm part of this group of people that consider
themselves addicts to open source. Big part of our work is created to be shared
by everyone, and like Spider-man's uncle said once... "With great power comes
great responsibility".

But what are those responsibilities we should take care of during our
development? What parts of our application should we really take care of and
which are more vulnerable over the time? Testing, documentation, clearness of
our code, abstraction, extension capabilities... we could talk about them all,
and for sure, each one can have enough material for an entire blog or a book.

In that case, I want to expose my personal experiences about what I learned over
the time by leading an open source project, several small open source bundles
and PHP libraries, and I want to do it by explaining how we should take care of
our Symfony bundles or PHP component dependencies.

## PHP Component Dependencies

When we talk about PHP components, we talk about framework agnostic packages,
only coupled to the language itself, in that case PHP, and to some other 
libraries. Having said that, we could start by trying to understand how the
components and bundles are split in some projects, for example
[Symfony](http://github.com/symfony/symfony) or
[Elcodi](http://github.com/elcodi/elcodi). Both projects have components and
bundles, providing the chance to all frameworks to work with their business
logic.

* So do we place all the business logic in components?
* Yes. The *why* of your project is placed in libraries. This will be your 
  service layer, and it should be covered with unit tests.
  
In regard to dependencies, components should only have dependencies on other
components. But how can I discover what packages I really depend on? In that 
case it's very simple, so we're not working with any kind of magic. Because our 
packages are simple PHP classes, checking the usage of all external classes
should be enough to know on which libraries we depend.

In that case, I always write code with `use` statements, so is much easier to
check my external classes usage. Just by checking the first lines of all my
classes I can guess what packages I should add in my composer.

``` php
namespace Elcodi\Component\User\EventListener;

use Symfony\Component\Security\Core\Event\AuthenticationEvent;
use Doctrine\Common\Persistence\ObjectManager;
use Elcodi\Component\Cart\Entity\Interfaces\CartInterface;
use Elcodi\Component\Cart\Wrapper\CartWrapper;
use Elcodi\Component\User\Entity\Interfaces\CustomerInterface;

/**
 * Class UpdateCartWithUserListener
 */
class UpdateCartWithUserListener
```

This piece of code makes you depend on four packages at least. Please, don't
focus on the versions, but only on the libraries.

``` json
"require": {
    "php": "*",
    
    "symfony/security": "*",
    "doctrine/common": "*",
    "elcodi/cart": "*",
    "elcodi/user": "*"
}
```

### Dependencies of Dependencies

Your package must manage ALL of its dependencies, even if they are as well
dependencies of your dependencies.

* Your package A uses B and C.
* Your package A requires B.
* Package B requires C.
* Then, your package A has both B and C. Enough.

Well, that's not true at all, because you cannot depend never of the
dependencies of your dependencies. Maybe now they require a package, but maybe
this package won't be required by your dependency anymore in the future. In that
case, even if you still need it, your package will disappear from your vendor
folder.

* Your package A uses B and C.
* Your package A requires B and C.
* Package B requires C.
* Then, your package A has both B and C.
* Package B does not require C anymore.
* You still have B and C.

Remember that... Require **ALL** your dependencies. All of them! That can make
the difference.

### Adapters

If our application is super decoupled from other libraries, and you have used
adapters for those integrations, then things change. Because the use of adapters
allows you to decouple from other packages literally, we should find another
mechanism to say... *hey, maybe you can depend on this package...*. Composer
proposes that mechanism, by using the `suggest` section.

``` json
"suggest": {
    "some/package": "Use ^2.5 for integration with Some Package"
}
```

Of course, this is not the only way of doing that. In this example we assume
that our package will offer all the adapters implementing an interface, and this
is just an option. In my case, I've been working for so long with a library
called [Gaufrette](https://github.com/KnpLabs/Gaufrette), and I really enjoy the
way these kind of packages work.

> Some of you could say... oh, but by doing that, then you're not defining
> dependencies but only suggestions (sounds the same as saying nothing, in
> fact), but when we define a requirement is when our package cannot exist
> without this package. When it is a MUST.

Other kind of implementations don't take into account the possibility of using the
`suggest` section in composer, because they don't really solve the dependencies
problem. This implementation forces having 1+`n` packages, the first one
containing the interface and the common content, and the other `n` containing
each specific implementation, all of them requiring the first one as a
dependency and the specific third-party package.

``` json
"require": {
    "myself/core": "*",
    "some/package": "^2.5 "
}
```

This is much more heavy to maintain, and only works if you only offer one port
with `n` adapters in your package. In the case of Gaufrette, this could be a
reality, so they only offer one port with `n` adapters, but of course, having
this structure is more difficult to maintain.

### Versions

That is a very complex topic. I will not talk about composer, but firstly, I
will share some basic concepts that are used a lot when defining dependencies
between packages.

* `~2.5.4` means equal and bigger than `2.5.4` but smaller than `2.6`
* `~2.5` means equal and bigger than `2.5.0` but smaller than `3.0.0`
* `^2.5.4` means equal and bigger than `2.5.4` but smaller than `3.0.0`
* `^2.5` means equal and bigger than `2.5.0` but smaller than `3.0.0`

The only thing I can say about that is that if your library aims to be usable by
a biggest community as possible, then please consider checking your dependencies
deeply, offering as much version-compatibility as possible.

The following composer requirements...

``` json
"require": {
    "php": "5.4.2",
    
    "symfony/security": "~2.7.3",
    "doctrine/common": "~2.7.3",
    "elcodi/cart": "~1.0.4",
    "elcodi/user": "~1.0.4"
}
```

are more restrictive than the following ones...

``` json
"require": {
    "php": "^5.3.9",
    
    "symfony/security": "^2.3",
    "doctrine/common": "^2.3",
    "elcodi/cart": "^1.0",
    "elcodi/user": "^1.0"
}
```

Of course, you should add requirement compliance as long as they really cover
your library needs. If you use Traits, then you should use `^5.4`, or if you're
using some Symfony features introduced in a specific version, then you become
dependent, at least, on this version

``` json
"require": {
    "php": "^5.4",
    
    "symfony/security": "^2.7",
    "doctrine/common": "^2.3",
    "elcodi/cart": "^1.0",
    "elcodi/user": "^1.0"
}
```

Consider as well the compatibility with new major versions, as soon as they
confirm their roadmap strategy and feature list. In that case, we could add
compatibility with Symfony `^3.0` if we have removed all deprecated elements
from old versions, or PHP `^7.0` if we don't use any `^5.6` deprecated function.

``` json
"require": {
    "php": "^5.4|^7.0",
    
    "symfony/security": "^2.7|^3.0",
    "doctrine/common": "^2.3|^3.0",
    "elcodi/cart": "^1.0",
    "elcodi/user": "^1.0"
}
```

This is very important, because if you don't offer this kind of compatibility,
no one using your package will be able to evolve properly, and when I mean
someone using your package I mean anyone using any package that, recursively,
uses your package.

That can be tons of projects.

## Symfony bundle dependencies

Once we have talked about PHP components, let's talk about Symfony bundles. This
is something much more complicated, because a Symfony Bundle is a PHP library
that is co-existing in a framework, so it is not as easy to check all our PHP
class dependencies.

The question we must ask ourselves when trying to resolve any Symfony bundle
dependency map is... *What do I really need to make this bundle work in any
Symfony project?*

### Other bundles

This is one of the things Symfony doesn't solve yet. How can a bundle depend on
another bundle, but not only in the composer layer but as well in the
application layer?

Well, there is a package for that (remember to **star** it if turns out useful
for you).

[Symfony Bundle Dependencies](https://github.com/mmoreram/symfony-bundle-dependencies)

This package allows you to create bundles with other bundle dependencies very
easily. By using this package you will be able to say... *okay composer,
download this bundles, I need them to instantiate my bundle... and Symfony
application, as soon you instantiate my bundle, please, install these other
bundles as well before* without any need to modify the kernel. Of course, this
is only possible if the project works with that package as well.

``` php
use Mmoreram\SymfonyBundleDependencies\DependentBundleInterface;

/**
 * My Bundle
 */
class MyBundle implements DependentBundleInterface
{
    /**
     * Create instance of current bundle, and return dependent bundle namespaces
     *
     * @return array Bundle instances
     */
    public static function getBundleDependencies(KernelInterface $kernel)
    {
        return [
            'Another\Bundle\AnotherBundle',
            'My\Great\Bundle\MyGreatBundle',
            new \Yet\Another\Bundle\YetAnotherBundle($kernel),
            new \Even\Another\Bundle\EvenAnotherBundle($kernel, true),
        ];
    }
}
```

If the project is not using this package, then the behavior of your bundle won't
change at all.

### DIC Services

To resolve all your bundle dependencies you need to take a look as well at your
Dependency Injection definition. Let's imagine we have a bundle with this
DIC definition.

``` yaml
services:

    my_service:
        class: My\Service\Class
        arguments:
            - @twig
            - my_other_service
            - @event_dispatcher
```

When Symfony tries to resolve this file, it needs as well all the definitions
of the arguments (dependencies). The bundles that have these definitions
automatically become your bundle dependencies.

The hard work here is to know which bundles have all these service definitions,
and that is not always that simple. In that case, for example... Which package
has the `@twig` service? We could think easily... well, `twig/twig` for sure has
the class we are injecting here.

And you're right, so if any of your classes, in that case `My\Service\Class`
needs a class from the package `twig/twig`, this package will have to be
required by your bundle.

But is that the real answer we need right now? Not at all. The question is not
which package provides me with the Twig class, but with the `@twig` service, and
this one is not `twig\twig` as this is only a PHP library, framework agnostic.

For this reason, we have a bundle called
[TwigBundle](http://github.com/symfony/TwigBundle). This bundle, as well as 
other needed things, creates a new service called `twig`. This Bundle is
required not only because we need the code under our vendor folder, but also
because it has to be instantiated when our bundle is instantiated.

``` json
"require": {
    "symfony/twig-bundle": "^2.7"
}
```

If you decide to work with the symfony bundle dependency package, then this code
is for you.

``` php
use Mmoreram\SymfonyBundleDependencies\DependentBundleInterface;

/**
 * My Bundle
 */
class MyBundle implements DependentBundleInterface
{
    /**
     * Create instance of current bundle, and return dependent bundle namespaces
     *
     * @return array Bundle instances
     */
    public static function getBundleDependencies(KernelInterface $kernel)
    {
        return [
            'Symfony\Bundle\TwigBundle\TwigBundle',
        ];
    }
}
```

If you don't use this package, then you should add the TwigBundle instance in
your AppKernel.

### Requiring the Framework

How about this services file? What dependencies do you think your package has?

``` yaml
services:

    my_service:
        class: My\Service\Class
        arguments:
            - @event_dispatcher
```

Some people can say if quickly... the EventDispatcher is a requirement... but we
have the same problem as before. The Event Dispatcher is a Symfony component,
and has nothing to do with the exposure of their classes in the Symfony
Framework dependency injection definition.

Symfony provides as well a bundle called
[FrameworkBundle](https://github.com/symfony/framework-bundle). Its mission is,
in addition to creating all the working environment for your project (the
framework itself), to expose all needed services to the DIC. One of them is the
Event Dispatcher from the component (if you check the composer.json file of that
bundle you will discover that the symfony/event-dispatcher package is a
requirement).

So, some of your bundle services should require as well this bundle.

``` json
"require": {
    "symfony/framework-bundle": "^2.7|^3.0"
}
```

This bundle is almost always required by all bundles (at least, it should), so
make sure you're really tolerant with its version, or you will make your 
bundle less usable than it could be.

### Symfony ^3.0.0

Many packages are actually requiring a very restrictive version of Symfony. This
fact has not been a problem during the latest 4 years, but nowadays Symfony
`v3.0.0` is going to be a reality soon, so all these packages need to make two
easy things

* Check if your bundle introduces a Symfony `~2.8` deprecated feature.
* If it does, update your bundle to avoid this deprecation
* Update your requirements to work as well with Symfony `^3.0`.

Check that your Symfony requirements then are still valid. For example:

``` json
"require": {
    "symfony/framework-bundle": "^2.2|^3.0"
}
```

Applying this new requirement with Symfony `^3.0` maybe you had to use a new
feature that was introduced in Symfony `^2.7.3`. In that case, your
composer.json is invalid, and if you have covered your class with tests and you
run your tests with `--prefer-lowest`, then you will have some fails there.

You will have to update your dependency properly.

``` json
"require": {
    "symfony/framework-bundle": "^2.7.3|^3.0"
}
```

## Development Dependencies

As you may already noticed, development dependencies are not loaded recursively.
This means that the `require-dev` block of your `require-dev` packages is
completely ignored.

In some way, this is great because you can define specifically what packages you
need for your development (testing, mostly), without being worried about all the
packages that require you.

In some other way, this can be bad... well, yes, you must know exactly all your
dependencies for testing (there should be only a few...), so in that case, just
make sure you know your application :)

### Requiring PHPUnit

And then the question is... should I require PHPUnit or other testing libraries,
as well as lints and formatters?

Again, some people will tell you... don't do that! Your development and testing
deployment will require more disk and more resources for composer. Well, sure,
but if you depend on the pre-installed PHPUnit version, then you can have some
trouble when testing.

``` json
"require-dev": {
    "fabpot/php-cs-fixer": "1.4.2",
    "mmoreram/php-formatter": "1.1.0",
    "phpunit/phpunit": "4.5.0"
},
```

I really need these versions. No others but these. For example, some lints can
add some logic, or even change it. Because we're responsible for our bundles,
components or code, we should trust as less as possible in what other people can
do to our dependencies (somehow we really trust a lot of packages by adding the
`^2.2` symbol, but in that case we do it not for us but for our users). In
testing mode, and because fortunately `require-dev` block is not recursive, we
can perfectly be restrictive with the version we want, and as long as we
want/need to change it... just do it :)

## Trust

Trusting open source is something you cannot do blindly. Your project is your
business, and you need to know that is safe from third party version errors and
issues.

If you trust a package, like I do for example in Symfony, then use the
semantic version notation in your requirements. Believe that the community will
never allow back compatibilities breaks, or they will fix them all as soon as
possible when introduced.

If you trust a library because it is tested, but you don't trust their
version policy, then just block the version (knowing that this restricts the
compatibility with other packages), or make some push to this community for a
really semantic version policy.

If you don't trust a library at all, then don't use it. That simple.

## Conclusion

So, that's it.

I highly recommend you, open source lover, to take as much care as possible of
your package dependencies. A healthy and useful package is a package used by
tons of people. Offer them some confidence and you will get a lot of feedback in
return.

Share your work as much as you can, and don't be afraid of your errors, they
will be your biggest reasons for being a better developer day after day, and
remember that all of us were inexperienced once.

*Error is first step to success*
