---
layout: post
title: "ParamConverter... mala idea"
date: 2014-05-05 13:18
comments: true
categories:
    - symfony
    - paramconverter
    - opensource 
---

Este problema ocurre en proyectos abiertos, open source. Para proyectos privados
donde Cart siempre será Cart, no creo que deba ser de su preocupación.

Cada vez tenemos más en mente trabajar con módulos que sean reutilizables. Las
buenas prácticas pasan por trabajar con interfaces y dar pie a que, el usuario
que tan amablemente está reutilizando nuestro código, pueda definir su dominio
según su especificación. Esto pasa, claro, por pensar que una entidad no siempre
será la misma entidad, el mismo objeto.

Me explicaré.

La buena práctica sugiere que cuando definimos una entidad con sus relaciones, 
utilizemos un ResolveTargetEntityListener. Lo que hace este listener es, una vez
se han definido las relaciones utilizando interfaces, se resuelven éstas y se
sustituyen por entidades reales ( evidentemente por entidades que implementan
tales interfaces ).

Cuando se sigue esta estrategia, es natural definir tanto el namespace como
el factory de la entidad que estás utilizando en un parámetro de bundle, por lo
que se puede sobrescribir fácilmente en cada uno de los proyectos.

Vemos un ejemplo.

``` yml

Elcodi\CartBundle\Entity\Cart:
    type: entity
    repositoryClass: Elcodi\CartBundle\Repository\CartRepository
    table: cart

    oneToOne:
        order:
            targetEntity: Elcodi\CartBundle\Entity\Interfaces\OrderInterface
            mappedBy: cart

```

En este caso, la relación con Order es débil, ya que la proponemos con cualquier
entidad que implemente OrderInterface. Solo hace falta que el 
ResolveTargetEntityListener proponga una implementación para OrderInterface, en
nuestro caso, `Elcodi\CartBundle\Entity\Order`.

En nuestro config, definimos los namespaces de las entidades con las que vamos a 
trabajar.

``` yml

parameters:

    #
    # Entities
    #
    elcodi.core.cart.entity.cart.class: Elcodi\CartBundle\Entity\Cart
    elcodi.core.cart.entity.order.class: Elcodi\CartBundle\Entity\Order

    #
    # Factories
    #
    elcodi.core.cart.factory.cart.class: Elcodi\CartBundle\Factory\CartFactory
    elcodi.core.cart.factory.order.class: Elcodi\CartBundle\Factory\OrderFactory

```

Este escenario anuncia lo que el título de este post enuncia. Hardcodear el
namespace de una entidad en alguna parte, rompe toda esta estrategia. Si
trabajamos de esta forma "abstracta" no nos podemos permitir el lujo de marcar
con sangre que una entidad siempre será de un tipo específico, y es, en 
realidad, lo que obliga el ParamConverter

``` php

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;

/**
 * @Route("/cart/{id}")
 * @ParamConverter("cart", class="ElcodiCartBundle:Cart")
 */
public function showAction(Post $post)
{
}

```

Proposición

Para que el ParamConverter funcionara de tal forma que pudiéramos salvar este
problema, deberíamos poder definir nuestra entidad con un parameter. Algo así:

``` php

use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
use Sensio\Bundle\FrameworkExtraBundle\Configuration\ParamConverter;

/**
 * @Route("/cart/{id}")
 * @ParamConverter("cart", class="%elcodi.core.cart.entity.cart.class%")
 */
public function showAction(Post $post)
{
}

```

Esto implica que se debería inyectar el Container entero en el servicio que 
trabaja los paramconverter de Doctrine, y que deberíamos checkear siempre si
el parámetro que se pasa es un namespace o un nombre de un parámetro.

Tal vez tendría sentido poder trabajar con ExpressionLanguage, pero el problema
es que añades una relación mas al bundle, por lo que no creo que les guste a los
autores del bundle ( SensioLabs ).

Con este post no quiero abrir un debate sobre si las annotations están bien o
no. Soy el primero que las he defendido en su momento, otorgando siempre su
utilidad en ciertos entornos y en ciertos casos. Ahora que empiezo a trabajar en
un entorno 100% OpenSource ya no valen tanto como antes.

Que creéis al respecto? Olvidaríais las annotations al 100 en entornos Open
Source?
