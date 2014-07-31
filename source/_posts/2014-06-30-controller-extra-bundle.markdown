---
layout: post
title: "ControllerExtraBundle, some useful Controller annotations"
date: 2014-06-30 17:03
comments: true
author: Marc Morera
categories:
    - symfony
    - annotations
    - ControllerExtraBundle

---
Few days ago, at the bimonthly meeting of Symfony2 Barcelona, attendees
began to discuss the annotations. Obviously, given that it is a
practice that goes against many many philosophies or architectural strategies ,
many people were against them, and some people where I include myself, were in
favor.

Being in favor of annotations does not mean wanting to use everywhere
but take them into account when the occasion required it is.

We could divide Symfony2 project into two blocks in this case. the
first is the model, a set of elements that implement our domain.
Doctrine has some annotations to define how this model maps to the
database, and I admit that at the first, when you are in the lower part
and elemental learning curve, are very positive elements for all developers.

However, at the time that you must have actual control of your domain, and
the time domain may be used in more than one project and more than
a framework, we have a problem. So, and I will not go into details,
I conclude that annotations never should be used in the domain model.

On the other hand, we have the application, the entry point. The controllers.

Let me ask you a question.

Do you think your application will change its framework someday? And given that
the answer is affirmative... Do you think you will maintain your controllers?

Both questions can be answered affirmatively, but is a very very rarely
situation where, surely, most of you will never have to manage. So I think that
we should thread Controllers in a very different way than Model.

IMHO, I am used to using annotations, because simplifies a lot some repetitive
processes, and gives me a very visual definition of what my action is responding
against, and what configuration are being applied.

Maybe is not the best of the options in some cases, but if the annotations is
well documented and defined, maybe could be useful in lot of cases.

In Symfony Controllers we have those annotations (Surely there are more, but I
consider that those are the most useful of them)

* ParamConverter - Good annotation. The code is a little bit messy but I am very
thankful. At the point I needed to define the namespace of the entity through a
container parameter, this annotations became useless.
* Method, Router y Template - Very useful for non-customizable projects. Worth
considering in our projects if we really do not need any kind of customization,
like custom routes.
* Secure - Static. Very useful if you accept your controllers will never change
its secure definition.

Given my little devotion for these controller annotations, some months ago I
decided to detect some needs I had in some personal projects, needs to repeat
some code once and again, code that had nothing to do with my business. I tried
to implement my annotations as flexible as possible.

For that I created [ControllerExtraBundle](https://github.com/mmoreram/ControllerExtraBundle).
This bundle, in addition to provide a set of Symfony2 annotations for
controllers, provides you a platform and a documentation to implement yourself
your own needs.

Actually this bundle is stable at version `v.1.1.2` and have this annotations

@Paginator
-------

[@Paginator Documentation](https://github.com/mmoreram/ControllerExtraBundle#paginator)


Allows you to inject a Doctrine Paginator given a route and a configuration. Is
very flexible but has to evolve even more.

``` php
/**
 * Simple controller method
 *
 * This Controller matches pattern /paginate/nb/{limit}/{page}
 *
 * Where:
 *
 * * limit = 10
 * * page = 1
 *
 * @PaginatorAnnotation(
 *      class = (
 *          factoryClass = "Mmoreram\ControllerExtraBundle\Factory\EntityFactory",
 *          factoryMethod = "create",
 *          factoryStatic = true
 *      ),
 *      page = "~page~",
 *      limit = "~limit~",
 *      orderBy = {
 *          { "x", "createdAt", "ASC" },
 *          { "x", "updatedAt", "DESC" },
 *          { "x", "id", "0", {
 *              "1" = "ASC",
 *              "2" = "DESC",
 *          }}
 *      },
 *      wheres = {
 *          { "x", "enabled" , "=", true }
 *      },
 *      leftJoins = {
 *          { "x", "relation", "r" },
 *          { "x", "relation2", "r2" },
 *          { "x", "relation5", "r5", true },
 *      },
 *      innerJoins = {
 *          { "x", "relation3", "r3" },
 *          { "x", "relation4", "r4", true },
 *      },
 *      notNulls = {
 *          {"x", "address1"},
 *          {"x", "address2"},
 *      }
 * )
 */
public function indexAction(Paginator $paginator)
{
}
```

@Entity
-------

[@Entity Documentation](https://github.com/mmoreram/ControllerExtraBundle#entity)

Generates a new entity instance, given an entity definition. This definition can
be given a namespace, a container parameter, a doctrine shortcut or a factory
definition.

``` php
<?php

use Mmoreram\ControllerExtraBundle\Annotation\Entity;
use Mmoreram\ControllerExtraBundle\Entity\User;

/**
 * Simple controller method
 *
 * This Controller matches pattern /user/edit/{id}/{username}
 *
 * @Entity(
 *      class = {
 *          "factory" = "my.factory.service",
 *          "method"  = "generate",
 *          "static"  = false,
 *      },
 *      name  = "user"
 * )
 */
public function indexAction(User $user)
{
    // $user is an empty instance
}
```

You can also, and since some days ago, not only inject an empty entity instance
but retrieve an existing mapped instance like ParamConverter does, but in a very
easy way.

``` php
<?php

use Mmoreram\ControllerExtraBundle\Annotation\Entity;
use Mmoreram\ControllerExtraBundle\Entity\User;

/**
 * Simple controller method
 *
 * This Controller matches pattern /user/edit/{id}/{username}
 *
 * @Entity(
 *      class = "MmoreramCustomBundle:User",
 *      name  = "user",
 *      mapping = {
 *          "id": "~id~",
 *          "username": "~username~"
 *      }
 * )
 */
public function indexAction(User $user)
{
    // $user is a mapped entity
}
```

@Form
-----

[@Form Documentation](https://github.com/mmoreram/ControllerExtraBundle#form)

Generates a simple FormType, FormInterface of FormView instance, given a form
definition. You can also configure it to create given a mapped or a new entity
instance and you can also handleRequest it and validate.

To create it given a mapped or a new entity instance, you can combine it using
both @ParamConverter or @Entity annotations.

``` php
<?php

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;
use Symfony\Component\Form\Form;

use Mmoreram\ControllerExtraBundle\Annotation\Form as AnnotationForm;
use Mmoreram\ControllerExtraBundle\Entity\User;

/**
 * Simple controller method
 *
 * @Route(
 *      path = "/user/{id}",
 *      name = "view_user"
 * )
 * @Entity(
 *      class = "MmoreramCustomBundle:User",
 *      name  = "user",
 *      mapping = {
 *          "id": "~id~"
 *      }
 * )
 * @AnnotationForm(
 *      class         = "user_type",
 *      entity        = "user"
 *      handleRequest = true,
 *      name          = "userForm",
 *      validate      = "isValid",
 * )
 */
public function indexAction(User $user, Form $userForm, $isValid)
{
}
```

@Flush
------

[@Flush Documentation](https://github.com/mmoreram/ControllerExtraBundle#flush)

Allows you to automate a flush every time a response is returned. There are
some real philosophies that say that a single flush should be executed always
at the end of the request.

``` php
<?php

use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;

use Mmoreram\ControllerExtraBundle\Annotation\Flush;
use Mmoreram\ControllerExtraBundle\Entity\Address;
use Mmoreram\ControllerExtraBundle\Entity\User;

/**
 * Simple controller method
 *
 * @ParamConverter("user", class="MmoreramCustomBundle:User")
 * @ParamConverter("address", class="MmoreramCustomBundle:Address")
 * @Flush(
 *      entity = {
 *          "user",
 *          "address"
 *      }
 * )
 */
public function indexAction(User $user, Address $address)
{
}
```

@JsonResponse
-------------

[@JsonResponse Documentation](https://github.com/mmoreram/ControllerExtraBundle#jsonresponse)

This annotation converts your controller return value in a single response,
using your data serialized with php `json_encode()`. This follows same way
@Template does.

IMHO, very useful.

``` php
<?php

use Mmoreram\ControllerExtraBundle\Annotation\JsonResponse;

/**
 * Simple controller method
 *
 * @JsonResponse
 */
public function indexAction(User $user, Address $address)
{
    return array(
        'This is my response'
    );
}
```

@Log
----

[@Log Documentation](https://github.com/mmoreram/ControllerExtraBundle#log)

Automatic log system for controller methods. You can define level of your log
and the moment you want to do it (before, after, both)

``` php
<?php

use Mmoreram\ControllerExtraBundle\Annotation\Flush;

/**
 * Simple controller method
 *
 * @Log(
 *      value   = "Executing index Action",
 *      level   = @Log::LVL_WARNING
 * )
 */
public function indexAction()
{
}
```