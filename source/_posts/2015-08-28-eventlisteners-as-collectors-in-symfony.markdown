---
layout: post
title: "EventListeners as collectors in Symfony"
date: 2015-08-28 12:04
comments: true
author: Marc Morera
categories: 
    - symfony
    - event listener
    - event dispatcher
    - collector
---
Some of my concerns during the last couple of years have been how to collect 
data from all installed bundles using available tools in Symfony packages. 
I say concerns because I don't really know if is there a tool for that.

Some people have told me that the 
[EventDispatcher component](http://symfony.com/doc/current/components/event_dispatcher/introduction.html) 
can do this work greatly, but then I have the same question once and again... is
this component designed for such?

Let's review some tiny concepts here.

## Event immutability

Try to think what really is an event. Something that happens. For example, I 
wake up. Once I wake up, an event is dispatched called `mmoreram.wake_up`. This
event, of course, is immutable. Nothing can change the fact that I woke up, so 
the event should be treated as an immutable object, with only reading actions.

One single property of the event is injected once is created. Did I wake up 
rested enough for a new crazy day with all cool guys from my office?

``` php
namespace Mmoreram;

use Symfony\Component\EventDispatcher\Event;

/**
 * Marc woke up event
 */
class MmoreramWakeUpEvent extends Event
{
    /**
     * @var boolean
     *
     * Marc woke up rested
     */
    private $rested;
    
    /**
     * Construct method
     *
     * @var boolean $rested Marc woke up rested
     */
    public function __construct($rested)
    {
        $this->rested = $rested;
    }
    
    /**
     * Get if Marc is rested enough
     *
     * @return boolean Marc is rested enough
     */
    public function isRested()
    {
        return $this->rested;
    }
}
```

Of course, no one should be able to change the value of rested, because no one
has the **power** to change the fact I woke up tired this night.

The main intention of an event is notify the world that something just happened,
so any extra implementation changing this paradigm should be avoided in order to
not corrupt the real meaning of the component.

Said that, and before continuing with the post... a question related to this
topic. Just make sure that you can take some time to think about that.

If we talk about decoupling between components... is the concept of `priority` 
helpful? If any actor must know the priorities of all listeners in order to know 
its own... then can we consider that all event listeners are really decoupled 
between them? And how bad is that?

## Collector

Let's figure out that the system need to collect some feelings when I wake up.
Let's figure out as well that we don't really care about how these feelings are
sorted, so the problem of priority is not a problem anymore: we can ignore it
completely.

We can change our event with this new implementation.

``` php
namespace Mmoreram;

use Symfony\Component\EventDispatcher\Event;
use Mmoreram\Feeling;

/**
 * Marc woke up event
 */
class MmoreramWakeUpEvent extends Event
{
    /**
     * @var boolean
     *
     * Marc woke up rested
     */
    private $rested;
    
    /**
     * @var Feeling[]
     *
     * Array of feelings
     */
    private $feelings;
    
    /**
     * Construct method
     *
     * @var boolean $rested Marc woke up rested
     */
    public function __construct($rested)
    {
        $this->rested = $rested;
        $this->feelings = [];
    }
    
    /**
     * Add new feeling
     *
     * @param Feeling $feeling New feeling to be added
     *
     * @return $this Self object
     */
    public function addFeeling(Feeling $feeling)
    {
        $this->feelings[] = $feeling;
        
        return $this;
    }
    
    /**
     * Get if Marc is rested enough
     *
     * @return boolean Marc is rested enough
     */
    public function isRested()
    {
        return $this->rested;
    }
    
    /**
     * Get all feelings
     *
     * @return Feeling[] Set of feelings
     */
    public function getFeelings()
    {
        return $this->feelings;
    }
}
```

Of course, in our domain we must dispatch this event one I really wake up (for
example a service called `MmoreramVitalActions`

``` php
use Symfony\Component\EventDispatcher\EventDispatcher;
use Symfony\Component\EventDispatcher\Event;
use Mmoreram\MmoreramWakeUpEvent;

/**
 * Marc vital actions
 */
class MmoreramVitalActions
{
    /**
     * Marc wakes up
     *
     * @return Feeling[] Set of feelings resulting of the action of waking up
     */
    public function wakeUp()
    {
        $rested = $this->didMarcRestedProperly();
        $eventDispatcher = new EventDispatcher();
        $event = new MmoreramWakeUpEvent($rested);
        
        $dispatcher
            ->dispatch(
                'mmoreram.wake_up',
                $event
            );
            
        return $event->getFeelings();
    }
    
    /**
     * Get if Marc rested properly
     *
     * @return boolean Marc rested properly
     */
    private function didMarcRestedProperly();
}
```

As you can see, after dispatching the event you should be able to get all
collected feelings. This means that the one in charge to fulfill this 
information related to my feelings should be any event listener interested in
adding it's own related feeling.

For example, an Event Listener will have the responsibility to add a feeling
related to the temperature of my room.

``` php
use Mmoreram\MmoreramWakeUpEvent;

/**
 * Marc wake up event listener related to temperature.
 * This class is intended to add a feeling depending on local temperature
 */
class TemperatureMmoreramWakeUpEventListener
{
    /**
     * Marc wakes up listener
     *
     * @param MmoreramWakeUpEvent $event Marc wake up event
     *
     * @return $this Self object
     */
    public function addTemperatureFeeling(MmoreramWakeUpEvent $event)
    {
        $feeling = $this->getTemperatureFeeling();
        $event->addFeeling($feeling);
        
        return $this;
    }
    
    /**
     * Get feeling related to the temperature
     *
     * @return Feeling Feeling related to temperature
     */
    private function getTemperatureFeeling();
}
```

Of course, you must add this event listener using tags in the Dependency 
Injection Symfony configuration.

[Using tags for our listeners definition](http://symfony.com/doc/current/reference/dic_tags.html)

At this point, you can see that maybe this is useful. This is a very easy and 
fast collector implementation, but not enough good. The event is not immutable
anymore and you can change if from any event dispatcher, very far from the real
intention of the component.

## Solution

I am using this approach in order to be as much pragmatic as possible. Of course
this works properly by adding an extra definition and documentation layer, but
I wonder if other people is concerned about that in Symfony.

I don't really think that yet another component called Collector would be 
necessary at all unless there is an abstraction between both components (they 
share some common things related to the fact of broadcasting and subscribing).

Of course, again, simple theory and personal thoughts brought to the community.
I will continue using this approach even knowing that should be solved using 
another one as long as people understand it and is easy to work with.

Feedback and people thoughts will be appreciated, as always :)