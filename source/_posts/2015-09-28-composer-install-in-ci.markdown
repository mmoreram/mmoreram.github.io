---
layout: post
title: "Composer install in CI"
date: 2015-09-28 16:53
comments: true
author: Marc Morera
categories: 
    - composer
    - travis
    - install
    - ci
---
If you have a project with a lot of dependencies, then you might now what I am 
talking about. We all love composer, but we all know as well that, at least with
current PHP versions, some composer.json are very heavy to compute and fulfill.
All this problems are reduced to time, and some CI engines, like Travis, only
allow a finite number of seconds (in Travis case, 20 minutes).

That's too much!

I found a solution (at least, I thought), but some days ago I saw that was not a
solution at all, but a workaround that only sometimes works properly. I will
explain exactly what's the point here.

### The problem

My `composer.json` is very big and composer needs too much time to compute all
the dependencies. Reducing dependencies is not an option at all, so the only way
of reducing dependencies is by doing some refactoring.

Any final project needs a lot of dependencies, and even if your `composer.json`
file is small, you may need a dependency with a lot of dependencies.

### My solution (the bad one...)

This solution is only wrong if you want to test your application under several
PHP versions. That's my case and could be yours...

Well. Computing the real dependencies in my environment seems a great solution,
right? I run `composer update` in my computer, I update the `composer.lock`
version in the repository, and then I only need to do `composer install`. What
I reduce here is the computing time of all recursive dependencies from 20+ 
minutes to less than 5 minutes.

That's great!

### Why this is a bad solution?

Some projects have decided to increase the minimum PHP dependency only
increasing the minor version of the package (and is not wrong, is not BC break).
If your project believes in Semantic Version (semver), is usual to find these 
pieces of composer blocks

``` json
"require": {
    "php": ">5.4",
    "symfony/symfony": "^2.7",
    "doctrine/orm": "^2.5",
    "some/package": "^1.3"
}
```

If my development environment uses PHP 5.5, then I will be able to work with
this composer requirements. Great. We update our dependencies, we update our
repository with the `composer.lock` file, and everything should work as
expected.

The problem here is that there is an scenario that we are not considering here,
and is that for sure, our `composer.lock` is the result of computing the
`composer.json` file in PHP 5.5, but this doesn't mean that same dependencies
will work as well in PHP 5.4.

Let's see the `some/package` composer definition in version 1.3.

``` json
"require": {
    "php": ">5.4",
    "whatever/whatever": *
}
```

And then, let's see the same composer file in version 1.4.

``` json
"require": {
    "php": ">5.5",
    "whatever/whatever": *
}
```

As long as we create the `composer.lock` file in our development environment
(remember, with PHP 5.5), we will use version 1.4 of package `some/package`, but
this package version is not compatible with PHP 5.4.

What will happens is that, when we do `composer install` in your CI, composer
will throw an Exception. And that's always bad news.

### The good solution

There's no good solution at all. In fact, there are only partial solutions, for
example generating the `composer.lock` file with the highest PHP version
allowed, but then, if you work with a dependency that forces a PHP version in
each version, this won't work at all.

``` json
"require": {
    "php": "5.4",
    "whatever/whatever": *
}
```

So, the only way of doing that is by using `composer update` in your CI
platform. The good point is that you **must** take care of your `composer.json`
file...

### For libraries

If you work with a library, then use the biggest dependency scope. This will
allow more users to use your library. Of course, you will increase the final
time of composer computation time, but is not your problem at all.

Of course, library composer dependencies should be as small as possible.

``` json
"require": {
    "php": ">5.4",
    "whatever/whatever": *
}
```

### For final projects

Final projects have the responsibility of reducing that scope, only allowing
explicitly highest versions. Of course, final projects can host big composer
structures (at least it's not a bad practice...), so in that case you will have
to work harder to reduce that file.

The problem here is not a problem at all, so it has no sense to test your final
application under several PHP versions, at least in your CI platform. Just test
it under your current PHP version, right?

``` json
"require": {
    "php": "5.5",
    "whatever/whatever": ^1.5.4
}
```
