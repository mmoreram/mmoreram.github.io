---
layout: post
title: "Cache clear en Symfony2"
date: 2014-01-03 15:11
comments: true
categories:
- Cache
- Symfony
---

Vacía la caché de Symfony, y la vuelve a crear (sino se especifica lo contrario)

``` bash
$ php app/console cache:clear [--no-warmup] [--no-optional-warmers]
```

Curioso comando existente en el framework Symfony2 desde sus inicios, y poco
utilizado por los proyectos. En realidad, he visto pocos proyectos que utilizen
este comando en los scripts de deploy, por lo que lo más habitual que veremos es
el clásico

``` bash
$ rm -Rf app/cache/*
```

Pero es correcta esta práctica? Aunque parezca la forma más ágil de limpiar
cache, tengamos en cuenta que Symfony2 tiene procesos internos que muchos de
nosotros desconocemos por completo. Cuando ejecutamos el comando para vaciar
caché ( con la opción `--no-warmup` ) no tan solo vaciamos el directorio caché
sino que el framework ejecuta el método `clear()` de todos los servicios que
implementan la interface 
`Symfony\Component\HttpKernel\CacheClearer\CacheClearerInterface` cuya
definición incorporen el tag `kernel.cache_clearer`.

En otras palabras.

``` yml
services:
    my_cache_clearer:
        class: Acme\MainBundle\Cache\MyClearer
        tags:
            - { name: kernel.cache_clearer }
```

Encontraréis mas información y especificación de estos tags
[aqui](http://symfony.com/doc/current/reference/dic_tags.html#kernel-cache-clearer).

## Reflexión

En realidad es importante fijarse en la opción `--no-warmup` ya que es la que
cambia el comportamiento del comando. Cuando trabajamos en entornos de
producción, necesitamos que, en tiempo de deploy, nuestra aplicación tenga el
mínimo tiempo de downtime. Esto se consigue reduciendo al máximo el tiempo en
que una caché está vacía.

### Tabla básica de tiempos

> * [1]   Repository status: S1, Cache status: C1
> * [1-3] Deploy S1 -> S2
> * [3-4] Clear cache C1 from /cache
> * [4-7] Warm up new cache C2 into /cache_new
> * [7-8] Swap cache C1 -> C2. Rename /cache_new to /cache
> * [8]   Repository status: S2, Cache status: C2

### Problema 1

Entre los tiempos 3 y 8, tenemos que /cache está vacía. Tenemos que tener en
cuenta que estamos en un entorno de producción, por lo que mientras hacemos el
proceso de deployment, nuestros servidores deben estar sirviendo peticiones en
todo momento.

Dado este caso, cada una de las peticiones se dará cuenta que la caché está
vacía, por lo que intentará volverla a montar desde cero. Deberíamos tener en
cuenta, entonces, que entre los tiempos 3 y 8, nuestro servicio estará caido.

De este análisis sacamos que debemos minimizar al máximo el tiempo pasado entre
el clear de la caché a invalidar y el switch a la nueva, o sea, reducir este
intervalo en el que nuestra caché está vacía.

Hace pocos dias propuse un
[PR a FrameworkBundle](https://github.com/symfony/symfony/pull/9930) para
gestionar esto.

Lo único que hace es cambiar el orden entre el warm up de la nueva caché y el
clear de la vieja.

> * [1]   Repository status: S1, Cache status: C1
> * [1-3] Deploy S1 -> S2
> * [3-6] Warm up new cache C2 into /cache_new
> * [6-7] Clear cache C1 from /cache
> * [7-8] Swap cache C1 -> C2. Rename /cache_new to /cache
> * [8]   Repository status: S2, Cache status: C2
>
> Downtime 2

En este punto podemos ver que la ventana que separa el clear de la caché antigua
y el switch con la nueva solo es de dos unidades de tiempo, entre los tiempos 6
y 8. Ahora se nos plantea otro problema.

### Problema 2

Hemos minimizado al máximo el tiempo de switch entre las dos caches, pero
seguimos teniendo una ventana de 5 unidades de tiempo entre que el repositorio
está en un estado S2 y la caché en un estado C1.

> Real downtime 5

En realidad, dado este análisis solo podríamos actuar de una sola forma, y es
trabajar con un deployment paralelo ( como hace el warmup del cache:clear ).
Esto significa que nunca deployamos sobre nuestra instalación activa

> * [1]   Repository status: S1, Cache status: C1 en /project
> * [1-8] Deployment Repository + cache:clear en /project_warmup
> * [8]   Repository status: S2, Cache status: C2 en /project_warmup
> * [8-9] Swap repository S1 -> S2. Rename /project to /project_warmup
>
> Downtime: 0

### Problema 3

En este caso es probable que nuestro proceso de deploy nos modifique elementos
externos, como puede ser ( y probablemente sea ) `mysql`. Entonces tendremos una
ventana bastante grande donde el modelo deployado en mysql que haga S2 no se
asemeje al modelo Doctrine de S1.

> Downtime: X

So what? Como lo hacéis vosotros? Ya es por sondear un poco hacia donde tira la
gente...
