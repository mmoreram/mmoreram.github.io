---
layout: post
title: "you probably need bundle dependencies"
date: 2016-02-09 12:23
comments: true
categories: 
---
This post tries to answer the Magnus Nordlander's
[blog post](https://blog.fervo.se/blog/2016/02/07/bundle-deps/), and to explain
why the [Symfony Bundle Dependencies](https://github.com/mmoreram/symfony-bundle-dependencies)
is not just a personal project to fulfill my bundles dependencies, but a
practice we should implement in all our Symfony bundles as well.

Believe me, I had a big post to explain why people really need this bundle, but
I think that you don't need these words, but a simple and real example.

Magnus, you're right. Maybe soft dependencies between bundles could be a good
option, but you know what? You know why Symfony is one of the biggest PHP
projects ever? Because Symfony understands the real user needs, and furthermore,
fulfills them the best way.

Why I tell this? Because your "You probably don't need bundle dependencies"
should be "You **really** need bundle dependencies, but you should work hard to
don't need them anymore".

Remember, software is real, with real developers, real projects and real needs.
We should take it in account as much as we can.

This is my example, from the most used bundle in the world, FOSUserBundle.

``` xml
<service id="fos_user.user_manager.default" class="FOS\UserBundle\Doctrine\UserManager" public="false">
    <argument type="service" id="security.encoder_factory" />
    <argument type="service" id="fos_user.util.username_canonicalizer" />
    <argument type="service" id="fos_user.util.email_canonicalizer" />
    <argument type="service" id="fos_user.object_manager" />
    <argument>%fos_user.model.user.class%</argument>
</service>
```

I want you to focus on one single line.

``` xml
<argument type="service" id="security.encoder_factory" />
```

We can discuss about how good or bad this is, but I really ensure you that you
will find this in the 99,99% of all bundles. So maybe we need to change our mind
and start doing decoupled bundles (not agree in fact, it depends on the case),
but right now, hard bundle dependencies (composer and kernel) is something that
should be covered the best we can.

About your last question, well, your libraries will require some other libraries
and composer will make it happen, with an update and it's autoloader. But your
bundles will probably require dependencies as well, with composer and as well
with a library like that, that will tell the Kernel witch bundle should be
instanced as well to complain services dependencies.