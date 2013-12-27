---
layout: post
title: "Inverse associations in Doctrine2 models"
date: 2013-10-09 11:37
comments: true
categories:
    - symfony
    - doctrine
    - relations
---
Lets take a look some basic example about simple Doctrine2 relation.

``` php
/**
 * Country
 *
 * @ORM\Entity
 * @ORM\Table(name="countries")
 */
class Country {

    /**
     * @var ArrayCollection
     *
     * @ORM\OneToMany(targetEntity="Province", mappedBy="country")
     */
    protected $provinces;

    /**
     * Constructor
     */
    public function __construct()
    {
        $this->provinces = new ArrayCollection;
    }

    /**
     * Add province
     *
     * @param Province $province Province to add
     *
     * @return Country self Object
     */
    public function addProvince()
    {
        $this->provinces[] = $province;

        return $this;
    }
}
```

``` php
/**
 * Province
 *
 * @ORM\Entity
 * @ORM\Table(name="provinces")
 */
class Province {

    /**
     * @var Country
     *
     * @ORM\ManyToOne(targetEntity="Country", inversedBy="provinces")
     * @ORM\JoinColumn(name="country_id", referencedColumnName="id", nullable=false)
     */
    protected $country;

    /**
     * Set Country
     *
     * @param Country $country Country
     *
     * @return Province self Object
     */
    public function setCountry(Country $country)
    {
        $this->country = $country;

        return $this;
    }

    /**
     * Get the country
     *
     * @return Country
     */
    public function getCountry()
    {
        return $this->country;
    }
}
```

The reason for this post is to try to understand the direct impact of making an inversed relationship when we allocate a new province in a country.
Given the model we have in the first instance, and given this piece of controller

``` php
$country = $this
    ->entityManager
    ->getRepository('AcmeCoreBundle:Country')
    ->findBy(1);

$province = new Province();
$province->setCountry($country);
$this->entityManager->persist($province);
$this->entityManager->flush();
```

In this case, when we assign the Country to the new Province, and given that the owning side of the association is Province, when you flush, the association persists in database, so that in future reference, we would have the desired results.
The "problem" exists because our `EntityManager` works with internal cache. If after this assignment, and in the same request, we need to return all the provinces of the Country, in particular the Country with id 1, the returned collection will not contain the new Province if this data is already cached. This is because we have not perfomed reversed assignment. There are two ways of solving this.

### Model owns the responsability of double assignment

We can resolve this giving model the responsability of double assignment.

``` php
/**
 * Province
 *
 * @ORM\Entity
 * @ORM\Table(name="provinces")
 */
class Province {

    /**
     * @var Country
     *
     * @ORM\ManyToOne(targetEntity="Country", inversedBy="provinces")
     * @ORM\JoinColumn(name="country_id", referencedColumnName="id", nullable=false)
     */
    protected $country;

    /**
     * Set Country
     *
     * @param Country $country Country
     *
     * @return Province self Object
     */
    public function setCountry(Country $country)
    {
        $this->country = $country;

        /**
         * We perform inversed assignment
         */
        $country->setProvince($this);

        return $this;
    }

    /**
     * Get the country
     *
     * @return Country
     */
    public function getCountry()
    {
        return $this->country;
    }
}
```

This has a good side and bad. On the one hand, is a completely transparent process so that the model handles internally to manage their relationships. It allows the driver to disengage completely from the model.
On the other hand, we must bear in mind that **will** always done this inverse relationship. This can be a plus for a project, unless handled very large amounts of data.

To give an example, we can imagine a Country with a million provinces. When charging Country in memory, while not run the `getProvinces ()` and since we work with lazy loading, we will have no memory problem. The problem comes when you add a new province to Country. As a collection, to add an item, doctrine do something like `getProvinces()` and then make `$provinces[] = $province;'.  Keep in mind that loading in memory up to 1 million entities without any need is a non desired behaviour.

### Controller ( each one ) owns the responsability of double assignment

In this case, controller need to perform inversed assignment.

``` php
$country = $this
    ->entityManager
    ->getRepository('AcmeCoreBundle:Country')
    ->findBy(1);

$province = new Province();
$province->setCountry($country);
$country->addProvince($province);
$this->entityManager->persist($province);
$this->entityManager->flush();
```

You will choose if is the best option, so you have an idea about if all provinces are used later. If are used and a million Provinces are loaded, you will have the same problem as first case, but has nothing to do about this...

My conclusion would be something like ... Know your model and scope, and give responsibilities accordingly.

What do you think?
