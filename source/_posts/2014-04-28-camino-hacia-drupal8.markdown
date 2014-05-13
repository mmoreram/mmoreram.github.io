---
layout: post
title: "Camino hacia Drupal8"
date: 2014-04-28 16:27
comments: true
alias: "/blog/2014/04/28/drupalcampmx/index.html"
categories: 
    - Drupal
    - Drupalcamp
    - México
    - Backdrop
---
Debo reconocer que 12 horas de avión son muchas horas. Sobretodo porque, aún
teniendo formas de distraerse ( Asumo de raiz que no se puede dormir... Podría
enumerar infinidad de razones... niños gritando, llorando o jugando, asientos
muy pequeños para la gente alta como yo... ) el reconocido JetLag desgasta toda
la semana posterior al viaje.

El tema es que, a pesar de todo, cuando la razón del viaje no es otra que
conocer compañeros nuevos, conocer una comunidad, una cultura, un país, una
gastronomía, unas costumbres... cuando la razón es mucho más que un simple
viaje de placer, estas horas pesadas pasan a ser lo de menos.

Hoy he vuelto de México tras una maravillosa semana en el DF, exactamente en el
DrupalCamp México 2014. Mi misión, hablar de Symfony a una comunidad que todavía
no lo tiene claro, viendo como su proyecto evoluciona a otro nivel.

Os pongo en contexto.

Drupal es un proyecto escrito en PHP4. Como wordpress, su core y su filosofía
se basas en un sistema de módulos instalables, reemplazando lo que nosotros
llamamos *Event Layer* por los ya mas que personalmente olvidados *Hooks*. 
El código está lleno de los llamados Drupalismos, un término creado y designado
a los *Coding Standards*, tanto en su forma como en su estructura en el cual se
basa todo el proyecto. 

Personalmente, muy mejorable.

Quiero aclarar de raiz que todo lo que añado a continuación es una simple
opinión. Mi punto de vista sobre el contexto social que se vive en todo el 
continente latinoamericano, y en toda la comunidad Drupal.

Siguiendo con el contexto, la versión estable de drupal actual es la 7. Drupal7
es, ha sido y será la última major release Drupal en lenguaje PHP4, 
así que en la nueva versión, Drupal8, algunos módulos de Symfony2 se añadirán 
como dependencias. Encontramos algo tan útil como el EventDispatcher ( En 
Drupal8 aún coexistirán tanto los Hooks como el EventDispatcher, y en la versión 
Drupal9, los hooks se eliminarán de forma completa y permanente ), el Routing, 
el HttpFoundation, el HttpKernel, el DependencyInjection...

Al parecer tal decisión ha dividido la comunidad en dos.

Por un lado tenemos la gente que no quiere pasar a Drupal8, alegando complejidad
demasiado elevada en la transición y un BC completo en ambas versiones. En este
colectivo se ha fundado el proyecto [Backdrop](http://backdropcms.org/), un CMS
evolucionado de Drupal7, con la misma filosofía de arquitectura, y con un cambio
en la forma de trabajar ( Sistema de releases, de features ) y por consiguiente,
con una comunidad estructurada.

Por otro lado tenemos a los que ven a Drupal8 como una posibilidad para
renovarse, para aventurarse en lo desconocido, aprender y resolver aquella deuda
técnica que han generado durante tanto años, apostando tal vez demasiado por la 
flexibilidad que propone Drupal desde hace tiempo.

Yo les conocí, a ambos, en el mismo lugar.

En este post quiero defender la evolución de un colectivo, la evolución de 
personas que muy honradamente quieren ganarse la vida, y a la par, aprender a 
programar utilizando las últimas tendencias en diseño, los patrones más 
innovadores, apostando por un modelo sostenible, colaborativo y 
completamente escalable.

Yo creo en el crecimiento personal como parte de un crecimiento superior, el de
una comunidad. Les conocí en el Camp y me transmitieron ganas de aprender, ganas
de adquirir conocimientos que, aparentemente, tanto cuesta adquirir. Tengo que
admitir que defender el primer colectivo es una cuestión de mediocridad muy
lícita ( como toda alternativa en el momento que, almenos a uno, le soluciona
un problema ). Defender el mero hecho de no evolucionar, me hace reflexionar
en que tal vez no se den cuenta que solo ellos son capaces de cambiar
el mundo, y estando en sus manos este cambio tan grande, deciden quedarse con lo
que "ya funciona". El "ya está bien, para que cambiarlo" no forma parte de
mi filosofía profesional, y por lo tanto, es imposible que llegue a lidiar
con ella.

Todo gran cambio requiere de un gran sacrificio. Todo gran aprendizaje requiere
un gran cambio, por lo que sacrificarte solo aporta nutrirse de nuevas opciones,
herramientas para el buen escultor, cuya ambición no es otra que tallar la 
figura perfecta.

Señores de *backdrop*, ustedes solo quieren seguir dotando a una comunidad 
gigante con un palo de madera para tallar figuras renacentistas. Me parece 
ridículo. Me parece prehistórico.

El hombre tuvo que aprender a hacer fuego de una simple rama, y ahora somos una
sociedad avanzada, en gran parte, por que podemos iluminar una habitación oscura
de la forma más primitiva.

Señores de *backdrop*, ustedes quieren seguir aceptando que PHP4 es válido, y no
lo es. Si fuera por PHP4, si fuera por la negación de la evolución cuando la POO
apareció antaño, hoy en día seríamos incapaces de hablar de Dependency 
Injection, de arquitecturas hexagonales y de patrones de diseño avanzados, cuya
función desde el mismo día de su proposición no ha sido otra que hacer 
evolucionar al pensador, al escultor, al programador.

Y creo que lo ha conseguido, con creces.

Y es que todo ser humano es simple. Si damos opciones, elige mal. Si proponemos
efectivamente el hecho de seguir siendo mediocre, el ser humano verá en esta
opción su zona de comfort, y no evolucionará.

Por qué no utilizan esta energía para mover masas? Por qué no utilizan esta 
fuerza que tan bien utilizan en quejarse, para ser el referente de un pueblo en
su camino hacia un nuevo punto de partida?

Finalmente, un mensaje al desarrollador Drupal indeciso. ¿Por qué teniendo 
herramientas para hacer catedrales magníficas, se empeñan en hacer chozas de 
madera? ¿Por qué no pasar al siguiente nivel?

Pues exactamente están en esta posición. ¿Acaso se creen que se hallarán solos en
este mar llamado desconocimiento? Tienen la oportunidad de crecer juntos como
comunidad hacia un objetivo común, mejorar, sea como sea. Yo no lo pensaría mas
de dos veces...

Sean valientes, interés es el pretexto del conocimiento y del éxito, así que les
animo a que den una oportunidad al progreso, a su capacidad, a que
Drupal8 y Symfony sean su tarjeta de embarque para su éxito personal y
profesional, así como un acceso directo a una comunidad proactiva y
colaborativa.

Aquí estaremos los que ya hemos sufrido para que su camino sea un poco más leve.