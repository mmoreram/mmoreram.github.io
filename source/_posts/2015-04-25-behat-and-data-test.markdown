---
layout: post
title: "Behat and data-test"
date: 2015-04-25 16:31
comments: true
author: Marc Morera
categories:
    - behat
    - behavioral
    - tests
---
Tests should be as robust as possible.

I think you will agree with me with that phrase. If your tests are too coupled
with your implementation, a simple modification of your code will need the 
modification of your tests, and that's so annoying, right?

I've using Behat for the last months in my projects, and as soon as you dominate
the tool it becomes really useful to make sure that any future refactoring or
change will break these user stories you have already defined and tested.

So, thinking about coupling I saw this method in Behat implementation.

``` php
/**
 * Presses button with specified id|name|title|alt|value.
 *
 * @When /^(?:|I )press "(?P<button>(?:[^"]|\\")*)"$/
 */
public function pressButton($button)
{
    //
}
```

What do you think? Do you think that this method helps people to really decouple
the tests from their implementation? I don't really think so... Let me explain.

### Design v Test

My question is... should the frontend of your website be aware of the how your
Behat tests are built? In my opinion, **nope**. 

Your tests should live in a simple layout on top of your application, emulating 
some cases and ensuring that your users will be able to do what they should be 
able to.

Said this, you cannot build your tests depending on final implementation. So,
whats the problem there?

### HTML Properties

Dear backend. Between you and me... we have nothing to do with some html 
properties, and you know that... This is the frontend world and we have nothing 
to say about that :) 

So why referring to html property `id` in your Behat cases? It has no sense 
indeed. You will need to change **all** your tests every time your frontend 
says... *refactoring time!!*, and we have no enough time for this, right?

So... first property strikethrough.

### Translations

Dear backend (again, yes). Between you and me... we have neither nothing to do
with translations, and we both know as well that translations is something 
really changeable (that means evolving... so yes, that are good news indeed), so
how about coupling your fantastic tests to translations? 

How do you really know that your submission button copy will be always `send`? 
What if someone thinks that is better `submit`? The point is that you **don't**
know that, and you will never do.

If you don't want to do that, please, don't use `title`, `alt` nor `value`. All
these html properties are very used to changing if you use them properly, so if 
you have your site in several countries with some modifications, you will not be 
able to reuse any scenario.

Bad choice again.

### Symfony Forms

We still have the `name` property, a very important property for forms and 
references inside your DOM. In fact, too much important to be an starting point 
for your test cases.

For example, you can fill a value in a Symfony Form input, but you know what?
Symfony Forms define themselves how their forms are named, in order to know how
to build them again after submitting them.

If you use `name` property, and for example you have different teams for 
developing your applications and for testing them, you will add an extra and 
useless coupling layer between them. This means more *points of failure* and, at
the end, less agility.

Not valid.

### So what?

Well, this is a *problem* really easy to solve. Have you ever meet the property 
`data-test`? You can build any property starting with `data-` and will be okay. 
So, in that case you can safely reference your elements using it.

* Your front-ends have nothing to do with it. They will see `data-test` and will
know that they belong to the testing layer. Then, they will ignore it, and even 
if they decide to refactor a page, they will preserve this property (if they 
can and want, of course) and your tests will not have any reason to expire.
* Your tests will have nothing to do with translation, product people and other
tactical nor strategical changes.
* People of your team will now have a unique way of referencing visible elements
of your application.

That's so nice!

### Implementation

Well, after this analysis, I propose to add `data-test` in all pre-defined 
selectors in order to allow people to uncouple from implementation.

``` php
/**
 * Presses button with specified id|name|title|alt|value|data-test.
 *
 * @When /^(?:|I )press "(?P<button>(?:[^"]|\\")*)"$/
 */
public function pressButton($button)
{
    //
}
```

Please, I would like to have some feedback, specially if you are used to working
with Behat or any kind of Behavioral Testing Tool.

Thanks and enjoy your day!