---
layout: post
title: "Flushing Doctrine2 entities"
date: 2013-10-11 10:11
comments: true
categories:
    - symfony
    - doctrine
    - flush
---
Given this example

``` php
$spain = new Country;
$spain->setName('spain');
$entityManager->persist($spain);

$france = new Country;
$france->setName('france');
$entityManager->persist($france);

$entityManager->flush();
```

When we flush without defining any kind of parameter, all entities managed by
EntityManager with changes will be flushed.  
To flush a specific entity managed by EntityManager we can just pass the entity
as a parameter in the flush method

``` php
$spain = new Country;
$spain->setName('spain');
$entityManager->persist($spain);

$france = new Country;
$france->setName('france');
$entityManager->persist($france);

/**
 * At this point, I only want to flush $spain
 */
$entityManager->flush($spain);
```

To flush an array of entities managed by EntityManager we can pass the array as
a parameter in the flush method

``` php
$spain = new Country;
$spain->setName('spain');
$entityManager->persist($spain);

$france = new Country;
$france->setName('france');
$entityManager->persist($france);

$germany = new Country;
$germany->setName('germany');
$entityManager->persist($germany);

/**
 * At this point, I only want to flush $spain
 */
$entityManager->flush(array(
    $spain,
    $france,
));
```

So, how about flushing an ArrayCollection of entities? Lets take a look at the
UnitOfWork code where contains this logic.

``` php
if ($entity === null) {
    $this->computeChangeSets();

} elseif (is_object($entity)) {

    $this->computeSingleEntityChangeSet($entity);
} elseif (is_array($entity)) {

    foreach ($entity as $object) {
        $this->computeSingleEntityChangeSet($object);
    }
}
```

Considering that ArrayCollection is an object, this code should be changed to
this code to also accept ArrayCollection and all iterable objects.

``` php
if ($entity === null) {
    $this->computeChangeSets();

} elseif (is_array($entity) || $entity instanceof IteratorAggregate) {

    foreach ($entity as $object) {
        $this->computeSingleEntityChangeSet($object);
    }
} elseif (is_object($entity)) {

    $this->computeSingleEntityChangeSet($entity);
}
```
