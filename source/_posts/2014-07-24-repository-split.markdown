---
layout: post
title: "Repository Split"
date: 2014-07-24 16:00
comments: true
author: Marc Morera
published: false
categories:
    - git split
---
Una de las técnicas mas chulas que tiene git es lo llamado `git subtree split`.
Os preguntareis para que sirve tal maravilla. Pues vamos a poner un ejemplo.

Escenario
---------

Tenemos un proyecto muy grande con distintos componentes. Imaginemos que una de
las particularidades de nuestro proyecto es que dichos componentes pueden ser
utilizados de forma individual, aunque tengan dependencias entre ellos. Vamos a
utilizar el proyecto Elcodi para ver un ejemplo real.

Una de las cosas en las que deberíamos pensar siempre en en el coste real de
mantener nuestro proyecto a medida que este se haga más y más grande. Al
principio podríamos llegar a pensar... - hombre, solo son 3 repositorios simples
, pues que sencillo es crear 3 repositorios en Github, cada uno de ellos
independientes, y ya los voy manteniendo. Por experiencia os digo... cuando
tenéis 20 repositorios es algo simplemente inconcebible. No tiene sentido.

La solución que hemos encontrado para Elcodi no es otra que agregar todos los
repositorios en un único repositorio.

``` yml
# elcodi/elcodi

README
composer.json
src/
    Elcodi/
        CoreBundle/
        ProductBundle/
        UserBundle
        ...
```

Este repositorio está disponible si tratas con el paquete entero, agregando a tu
composer la dependencia `elcodi/elcodi`