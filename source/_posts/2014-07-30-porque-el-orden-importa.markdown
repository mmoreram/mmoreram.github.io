---
layout: post
title: "Porque el orden importa"
date: 2014-07-30 11:07
comments: true
author: Marc Morera
categories:
    - php
    - standards
---
La verdad es que reconozco que soy un poco neurótico con el código. Y es que en
mi caso me gusta definir la belleza de mis lineas no tan solo en el aspecto
funcional y en el aspecto de arquitectura, sino en el aspecto visual. Y en
realidad considero que descuidar este aspecto es un problema muy grande a la
larga.

Tratemos de pensar en la cantidad de horas que estamos ante estas lineas de
código, y luego tratemos de analizar el impacto que estas tienen en nuestro día
a día. ¿No creéis que importa el hecho de que esté ordenado, limpio y tratable?

Pues bien, es para esto que, y dado que llevo un tiempo con la manía de ordenar
los namespaces de mis ficheros php, he decidido empezar el proyecto
[php-formatter](https://github.com/mmoreram/php-formatter).

``` bash
Console Tool

Usage:
  [options] command [arguments]

Options:
  --help           -h Display this help message.
  --quiet          -q Do not output any message.
  --verbose        -v|vv|vvv Increase the verbosity of messages
  --version        -V Display this application version.
  --ansi              Force ANSI output.
  --no-ansi           Disable ANSI output.
  --no-interaction -n Do not ask any interactive question.

Available commands:
  help       Displays help for a command
  list       Lists commands
use
  use:sort   Sort Use statements
```

Lejos de ser un analizador de PSR, cuya función ya cumple a la perfección el
proyecto de Fabien Potencier [php-cs-fixer](https://github.com/fabpot/php-cs-fixer),
se trata de añadir, en modo comando, algunas funcionalidades tediosas que para mí
son importantes, como es el caso de los Use Statements ( Poder ordenarlos por
grupos, alfabéticamente o por longitud, ascendiente o descendiente... )

De momento solo hay esta funcionalidad, y aún le faltan un par de opciones, pero
no dudéis en proponer comandos que nos puedan ser útiles a todos.

Os invito a que tengáis reglas estructurales y visuales en vuestros proyectos,
ya que ayudan un poco a que el código quede un poco más estandarizado con el
paso del tiempo, aunque sea a nivel visual.

Vuestros ojos os lo agradecerán.