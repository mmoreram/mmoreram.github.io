---
layout: post
title: "Elcodi RuleBundle"
date: 2014-05-13 20:12
comments: true
categories:
    - Elcodi
    - Rule
    - ExpressionLanguage
    - Ecommerce
---
An ExpressionLanguage based Rule engine
---------------------------------------

Ésta implementación trabaja con el ExpressionLanguage de Symfony.

Para instalarlo o entender como funciona, así como para descubrir la syntaxis de las Expressions puedes utilizar la documentación oficial del Componente [aquí](http://symfony.com/doc/current/components/expression_language/index.html)

Our needs
---------

En realidad podríamos que estamos ante uno de los mayores problemas que ha sufrido cualquier programador que se haya dedicado mínimamente al e-commerce: Las reglas de acción. El ejemplo más significativo son los cupones. Cuándo un cupón es válido? Y dado que hay una cantidad bestial de distintos cupones y distintas lógicas de aplicación, y dado que éstas son distintas y própias de cada uno de los proyectos... como se puede abstraer la lógica para que se pueda hacer algo "neutral" y lo suficientemente
flexible para que todo el mundo lo pueda utilizar?

Pues esta es una de nuestras preocupaciones principales, ya que la solución a esta pregunta, de rebote, nos soluciones otros problema paralelos, como por ejemplo cuando se aplican ciertas acciones, o hasta generar un eventDispatcher muy flexible, que active cierto evento en cierto punto de la request, si aplica un conjunto de condicionales.

Our proposal
------------

[Elcodi RuleBundle](https://github.com/elcodi/RuleBundle)

Nuestra propuesta no es demasiado compleja.

Partimos de un modelo completamente desacoplado y standalone. El motor de reglas proporciona una gestión de las mismas independientemente de donde se quieran utilizar, ya que la única misión para este módulo es devolver true o false.

En este caso hemos tratado con dos tipos de reglas, las Rules y las RuleGroups. El sentido de la primera es simple, una regla simple que, evaluada, devuelve un booleano. El sentido de la segunda es, en cambio, agrupar Rules, de tal forma que si decidimos aplicar un conjunto de reglas unidas, podamos trabajar con el conjunto directamente, y asociarlo a un comportamiento.

Dado este modelo, es bastante evidente que requiere de un Composite Pattern para que sea más fácil trabajar, ya que éste nos proporciona una sola entidad, independientemente que sea Rule o RuleGroup, y le da un comportamiento común a ambos que es al fín y al cabo el mas importante: Evalúate.

Composite Pattern using Doctrine
--------------------------------

La primera pregunta que nos viene a la cabeza es... De que se trata el Composite Pattern?

![Composite pattern](http://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/Composite_UML_class_diagram_%28fixed%29.svg/900px-Composite_UML_class_diagram_%28fixed%29.svg.png "composite pattern")

En éste caso tenemos que tanto Leaf como Composite son dos entidades distintas. Leaf sería nuestro Rule y Composite sería nuestro RuleGroup. Podemos ver que, como ambos extienden de un objeto abstracto llamado Component, se puede forzar que ambos tengan un comportamiento común. En esta gráfica, el abstracto define un método y lo implementa, dando un comportamiento por defecto y dando la oportunidad de sobreescribir éste comportamiento. En nuestro caso, `operation()` responde a la necesidad de devolver todas las expressions del objeto (En el caso de Rule, una sola expression, en el caso de RuleGroup, todas las expressions de sus hijos), por lo que se ha definido el método como abstracto, así cada entidad debe implementar el método a su modo.

Finalmente, y lo que define realmente a éste patrón, es que Composite ( RuleGroup ) tiene un conjunto de hijos ( en nuestro caso Rules ) que tanto pueden ser del tipo Rule o del tipo RuleGroup, por lo que la relación se hace directamente con el Abstracto.

Esta implementación permite que, cuando se pide las Expression de un RuleGroup, ésta te devuelva la de todos sus hijos, sean Rule o RuleGroup, ya que ambos elementos implementan el mismo método.

Doctrine te permite hacer ésta implementación utilizando el STI ( Single Table Inheritance ) o el CTI ( Class Table Inheritance ). En éste caso, y para ahorrarnos demasiadas joins, hemos tratado la implementación con Single Table.

Definimos la entidad Rule

``` php
<?php

/**
 * This file is part of the Elcodi package.
 */

namespace Elcodi\RuleBundle\Entity;

/**
 * Class Rule
 */
class Rule extends AbstractRule implements RuleInterface
{
    /**
     * @var ExpressionInterface
     *
     * Expression
     */
    protected $expression;

    /**
     * Specific class parameters, getters and setters
     */

    /**
     * Return all object contained expressions
     *
     * @return ArrayCollection Collection of expressions
     */
    public function getExpressionCollection()
    {
        return new ArrayCollection(array($this->getExpression()));
    }
}
```

Definimos la entidad RuleGroup

``` php
<?php

/**
 * This file is part of the Elcodi package.
 */

namespace Elcodi\RuleBundle\Entity;

/**
 * Class RuleGroup
 */
class RuleGroup extends AbstractRule implements RuleGroupInterface
{
    /**
     * @var Collection
     *
     * Rules
     */
    protected $rules;

    /**
     * Specific class parameters, getters and setters
     */

    /**
     * Return all object contained expressions
     *
     * @return ArrayCollection Collection of expressions
     */
    public function getExpressionCollection()
    {
        $expressions = array();

        /**
         * @var AbstractRuleInterface $rule
         */
        foreach ($this->getRules() as $rule) {

            $expressions = array_merge(
                $expressions,
                $rule->getExpressionCollection()->toArray()
            );
        }

        return new ArrayCollection($expressions);
    }
}
```

Y finalmente la clase abstracta

``` php
<?php

/**
 * This file is part of the Elcodi package.
 */

namespace Elcodi\RuleBundle\Entity\Abstracts;

/**
 * Class AbstractRule
 */
abstract class AbstractRule extends AbstractEntity
{
    /**
     * Specific class parameters, getters and setters
     */

    /**
     * Return all object contained expressions
     *
     * @return Collection Collection of expressions
     */
    abstract public function getExpressionCollection();
}
```

Ambas clases implementan a su modo el método `getExpressionCollection()` por lo que cuando se trate con un ArrayCollection devuelto por Doctrine, todas las instancias tendrán el método.

Una vez tenemos las clases, debemos mapearlas a Doctrine.

Mapeamos la clase Rule, con una relación unidireccional con Expression

``` yml
Elcodi\RuleBundle\Entity\Rule:
    type: entity
    repositoryClass: Elcodi\RuleBundle\Repository\RuleRepository
    table: rule

    oneToOne:
        expression:
            targetEntity: Elcodi\RuleBundle\Entity\Expression
            cascade: [ all ]
```

Mapeamos la clase RuleGroup, con una relación ManyToMany unidireccional ( No nos interesa saber `parents()` ) con AbstractRule

``` yml
Elcodi\RuleBundle\Entity\RuleGroup:
    type: entity
    repositoryClass: Elcodi\RuleBundle\Repository\RuleGroupRepository
    table: rule_group

    manyToMany:
        rules:
            targetEntity: Elcodi\RuleBundle\Entity\Abstracts\AbstractRule
            joinTable:
                name: rule_group_rule
                joinColumns:
                    rule_group_id:
                        referencedColumnName: id
                inverseJoinColumns:
                    rule_id:
                        referencedColumnName: id

```

Y finalmente generamos el mapeo de la clase abstracta

``` yml
Elcodi\RuleBundle\Entity\Abstracts\AbstractRule:
    type: entity
    repositoryClass: Elcodi\RuleBundle\Repository\AbstractRuleRepository
    inheritanceType: single_table
    discriminatorColumn:
        name: discr
        type: string
    discriminatorMap:
        rule: Elcodi\RuleBundle\Entity\Rule
        rule_group: Elcodi\RuleBundle\Entity\RuleGroup
    fields:
        ...
```

Esta configuración creará una sola tabla, con un campo dedicado a decirle a Doctrine de que tipo es la fila en cuestión.

Llegados a éste punto, tenemos una implementación para poder asignar a cualquier entidad una relación con Rules, teniendo solucionado, almenos a nivel de dominio y de implementación el concepto "Regla de Reglas", pero ahora falta solucionar el cómo se ejecutan éstas reglas y el contexto en el que lo hacen.

Context configuration and customization
---------------------------------------

Definamos contexto en el que una regla se ejecuta como el conjunto de elementos en el que la regla tiene acceso en el momento de la ejecución.

Ya que el motor trabaja utilizando el ExpressionLanguage de Symfony, hemos utilizado su forma de contextualizar una evaluación.

``` php
$language = new ExpressionLanguage();

class Apple
{
    public $variety;
}

$apple = new Apple();
$apple->variety = 'Honeycrisp';

$this->assertTrue($language->evaluate(
    'fruit.variety === "Honeycrisp"',
    array(
        'fruit' => $apple,
    )
));
```

En este caso, ejecutamos la expresión "fruit.variety" en un contexto donde `fruit` es nuestra instancia de `Apple`, por lo que éste assert resolvería como true.
En el caso siguiente, y teniendo la misma expresión, al tener una contextualización distinta, no tendríamos el resultado esperado, pues devolvería `false`

``` php
$language = new ExpressionLanguage();

class Apple
{
    public $variety;
}

$apple = new Apple();
$apple->variety = 'Arlet';

$this->assertTrue($language->evaluate(
    'fruit.variety === "Honeycrisp"',
    array(
        'fruit' => $apple,
    )
));
```

Hay dos tipos de contextualizaciones, las que responden a una request ( En toda las evaluaciones dada una request se setean unos valores globales, independientemente de la ejecución ) y las que responden a cada llamada del manager.

En éste caso se trata de una contextualización de llamada, pues en ésta evaluación, `customer` responde a $myCustomer.

``` php

/**
 * Rule with code "somecode" has this expression assigned: 
 * "customer.id in [1..100]"
 */ 
$myCustomer = ...;
$ruleManager = $this->container->get('rule_manager');
$result = $ruleManager->evaluateByCode('somecode', array(
    'customer'  =>  $myCustomer
));
```

Es posible pero que tengamos servicios, por ejemplo, a los que queramos acceder de forma reiterada, y que haya una forma independiente de acceder a ellos, por ejemplo, el `entity_manager`.

``` php

/**
 * Rule with code "somecode" has this expression assigned: 
 * "customer_wrapper.getCustomer().getId() in [1..100]"
 */

$ruleManager = $this->container->get('rule_manager');
$result = $ruleManager->evaluateByCode('somecode');
```

`customer_wrapper` en realidad es un servicio que, a su forma, nos instancia un Customer recuperable mediante el método `getCustomer`. No debemos pasarlo cada vez que queremos evaluar una Expresión, así que podemos considerarla como parte del contexto global.

Como podemos añadir valores a éste contexto global? Pues implementando un `ContextConfigurationInterface`.

``` php
<?php

/**
 * This file is part of the Elcodi package.
 */

namespace Elcodi\RuleBundle\Configuration;

use Doctrine\Common\Persistence\ObjectManager;

use Elcodi\RuleBundle\Configuration\Interfaces\ContextConfigurationInterface;
use Elcodi\RuleBundle\Services\Interfaces\ContextAwareInterface;

/**
 * Class ContextConfiguration
 */
class ContextConfiguration implements ContextConfigurationInterface
{
    /**
     * @var ObjectManager
     *
     * Object manager
     */
    protected $objectManager;

    /**
     * Construct method
     *
     * @param ObjectManager $objectManager Object manager
     */
    public function __construct(ObjectManager $objectManager)
    {
        $this->objectManager = $objectManager;
    }

    /**
     * @param ContextAwareInterface $contextAware
     */
    public function configureContext(ContextAwareInterface $contextAware)
    {
        $contextAware->addContextElement('manager', $this->objectManager);
    }
}
```

Podemos observar que en este caso estamos contextualizando todo el gestor de expresiones con el `EntityManager` respondiendo al valor `manager`, asi que si en una Expression tenemos a expresión `manager.getRepository("...")` podremos acceder perfectamente al EntityManager.

Finalmente, y para terminar de añadir éste valor al contexto, debemos definir ésta clase como servicio utilizando un tag específico.

``` yml
services:
    elcodi.core.rule.configuration.context:
        class: %elcodi.core.rule.configuration.context.class%
        arguments:
            entity_manager: @doctrine.orm.entity_manager
        tags:
            - { name: elcodi.rule_context_configuration }
```

ExpressionLanguage configuration and cuztomization
--------------------------------------------------

Otra de las facilidades que nos brinda el ExpressionLanguage es la capacidad de crear funciones. En este caso podemos agregar nuevas funcionalidades al ExpressionLanguage del sistema de Reglas implementando `ExpressionLanguageConfigurationInterface`

``` php
<?php

/**
 * This file is part of the Elcodi package.
 */

namespace Elcodi\RuleBundle\Configuration;

use Symfony\Component\DependencyInjection\ContainerInterface;

use Elcodi\RuleBundle\Configuration\Interfaces\ExpressionLanguageConfigurationInterface;
use Elcodi\RuleBundle\Services\Interfaces\ExpressionLanguageAwareInterface;

/**
 * Class ExpressionLanguageConfiguration
 */
class ExpressionLanguageConfiguration implements ExpressionLanguageConfigurationInterface
{
    /**
     * @var ContainerInterface
     *
     * Container
     */
    protected $container;

    /**
     * Construct method
     *
     * @param ContainerInterface $container Container
     */
    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    /**
     * Configures expression language
     *
     * @param ExpressionLanguageAwareInterface $expressionLanguageAware Expression Language aware
     */
    public function configureExpressionLanguage(ExpressionLanguageAwareInterface $expressionLanguageAware)
    {
        $expressionLanguage = $expressionLanguageAware->getExpressionLanguage();
        $expressionLanguage
            ->register('service', function ($arg) {
                return sprintf('$this->get(%s)', $arg);
            }, function (array $variables, $value) {
                return $this->container->get($value);
            });

        $expressionLanguage
            ->register('parameter', function ($arg) {
                return sprintf('$this->getParameter(%s)', $arg);
            }, function (array $variables, $value) {
                return $this->container->getParameter($value);
            });
    }
}
```

Vemos que hemos añadido el mismo comportamiento que tiene el ExpressionLanguage cuando se utiliza en las definiciones del DependencyInjection. Se agregan dos funciones nuevas y ambas requieren del container entero, por lo que se tiene que definir ésta clase como servicio, añadiendo de nuevo un tag específico.

``` yml
services:
    elcodi.core.rule.configuration.expression_language:
        class: %elcodi.core.rule.configuration.expression_language.class%
        arguments:
            service_container: @service_container
        tags:
            - { name: elcodi.rule_expression_language_configuration }

```

De esta forma, podríamos ejecutar ésta expresión `service("my_service")->getValue(parameter("my_value"))` sin tener una Exception del tipo SyntaxException.

El RuleManager controla cuando una expresión tiene una exceptión del tipo SyntaxException ya que tiene todo el contenido de evaluación dentro de un try/catch. El sentido de ésto es que en caso que una expresión tenga un error porque, o bien el modelo ha cambiado o la propia implementación, debería seguir funcionando, evidentemente devolviendo `false` como resultado.