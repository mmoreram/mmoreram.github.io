---
layout: post
title: "Re-thinking Event Listeners"
date: 2015-08-20 12:27
comments: true
author: Marc Morera
categories: 
    - Symfony
    - Event Listener
---
Let's talk about Event Listeners. 
Do you know what an Event Listener is?

Well, if you are used to working with Symfony, then you should know what is 
intended for. If you don't, don't hesitate to take a look at the Symfony 
documentation.

This post aims to start a small discussion about how an Event Listener should
look like if we really want to keep things uncoupled. 

## TL;DR

* If we place our business logic inside the Event Listeners, we will not be able 
to use this logic from other points, for example sending a "Order created" email 
by hand from our Admin using a simple button.
* What we could do is place ALL the logic inside a service that ONLY sends that
email, exposing a simple and documented api.
* Then, considering that this service is injectable using any Dependency 
Injection implementation, we can inject it into the Event Listener.
* So, EventListeners are only a point of entry to your service layer

## Using Event Listeners

To understand what I am talking about, let's use an small example to make things 
more clear...

```
Scenario: You buy something in an e-commerce, so, internally, your Cart became 
an Order. Of course, and because user experience is important in that cases, you 
want to send an email to the user with some Order information, so you need to 
send an email to the Customer.

Problem: You want to create this feature in a very decoupled way, of course. The
e-commerce framework provides you that way, by proposing you an event once the
Order is created. You can access the Order itself and the Customer.

Solution: Create a new Event Listener object, subscribed to this event called
order.oncreate, and sending that email.
```

Let's see a small example about how this Event Listener should look like. We 
will follow the simple way, only focusing about sending that email in a 
decoupled way with the action.

``` php
class OrderEmailEventListener
{
    /**
     * We send an email to the Customer once an order is created
     *
     * @param OrderOnCreatedEvent $event Event
     */
    public function sendEmail(OrderOnCreatedEvent $event)
    {
        $order = $event->getOrder();
        $customer = $event->getCustomer();
        
        /**
         * Send the email
         */
    }
}
```

And we could use this configuration in our bundle.

``` yaml
services:

    #
    # Event Listeners
    #
    project.event_listener.order_created_email:
        class: My\Bundle\EventListener\OrderCreatedEmailEventListener
        tags:
            - { name: kernel.event_listener, event: order.oncreate, method: sendEmail }
```

So this could be a simple implementation. I've been using it since the beginning
of times as this could be considered a good practice. But, during this time
doing more and more Event Listeners, some questions have come to my mind.

## Decoupling from the Event

Let's consider that our project has an admin panel. Of course, we should be able
to send this email any time we need (for example, our email server was down 
during the order conversion and we must re-send it). Is this possible with this
implementation?

Yes. Let's do this considering that we have injected our EventListener and this
one is accessible locally!

```php
$event = new OrderCreatedEmailEventListener(
    $order,
    $customer
);
$orderCreatedEmailEventListener->sendEmail($event);
```

Well, this piece of code will really send the email... but is this 
implementation enough right? I don't think so...

This Event should only be dispatched when the real event happens. It has no 
sense to create a new `OrderEmailEventListener` instance without using the
event dispatcher. This means that, indeed, any Order has been created.

So first of all, creating a new Event out of turn, is not a good practice at 
all.

For solving this, We could do that.

``` php
class OrderCreatedEmailEventListener
{
    /**
     * We send an email to the Customer once an order is created
     *
     * @param OrderOnCreatedEvent $event Event
     */
    public function sendEmail(OrderOnCreatedEvent $event)
    {
        $order = $event->getOrder();
        $customer = $event->getCustomer();
        
        $this->sendOrderCreatedEmail(
            $order,
            $customer
        );
    }
    
    /**
     * We send an email to the Customer once an order is created, given the 
     * order and the customer
     *
     * @param OrderInterface    $order    Order
     * @param CustomerInterface $customer Customer
     */
    public function sendOrderCreatedEmail(
        OrderInterface $order,
        CustomerInterface $customer
    ) {
        /**
         * Send the email
         */
    }
}
```

And then, we could do this

```php
$orderCreatedEmailEventListener->sendOrderCreatedEmail(
    $order,
    $customer
);
```

Much better, right?
But is this good enough? No is not.

## Decoupling from the Listener

We are using an instance of an Event Listener to send an email. Our analysis 
could be exactly the same than before... Should we use an Event Listener even
when an event is not dispatched?

No we should not.

An Event Listener is an event listener. Listens one event, and that should be 
all its work. So, we should never inject any event listener, anywhere. Let's do
some refactor here!

First of all, let's isolate our business logic in a new service. This service 
will **only** do one thing; sending this email.

``` php
class OrderCreatedEmailSender
{
    /**
     * We send an email to the Customer once an order is created, given the 
     * order and the customer
     *
     * @param OrderInterface    $order    Order
     * @param CustomerInterface $customer Customer
     */
    public function sendEmail(
        OrderInterface $order,
        CustomerInterface $customer
    ) {
        /**
         * Send the email
         */
    }
}
```

This service has only one mission. Sending this email, no matter what event
executes it, no matter its environment. So if we take a look at what the Event
Listener implementation should look like now...

``` php
class OrderCreatedEmailEventListener
{
    /**
     * @var OrderCreatedEmailSender
     *
     * Order created email sender
     */
    private $orderCreatedEmailSender;
    
    /**
     * Constructor
     *
     * @param OrderCreatedEmailSender $orderCreatedEmailSender
     */
    public function __construct(OrderCreatedEmailSender $orderCreatedEmailSender)
    {
        $this->orderCreatedEmailSender = $orderCreatedEmailSender;
    }

    /**
     * We send an email to the Customer once an order is created
     *
     * @param OrderOnCreatedEvent $event Event
     */
    public function sendEmail(OrderOnCreatedEvent $event)
    {
        $order = $event->getOrder();
        $customer = $event->getCustomer();
        
        $this
            ->orderCreatedEmailSender
            ->sendEmail(
                $order,
                $customer
            );
    }
}
```

Finally, we should refactor as well the way we have defined our service in the
DependencyInjection config file.

``` yaml
services:

    #
    # Business layer
    #
    project.business.order_created_email_sender:
        class: My\Bundle\Business\OrderCreatedEmailSender

    #
    # Event Listeners
    #
    project.event_listener.order_created_email:
        class: My\Bundle\EventListener\OrderCreatedEmailEventListener
        arguments:
            - @project.business.order_created_email_sender
        tags:
            - { name: kernel.event_listener, event: order.oncreate, method: sendEmail }
```

And that's all.
This example is so easy and simple, but I am sure that if you take a look at 
your project, you will find a lot of logic inside your Event Listeners. Maybe
could be a good idea start moving all this logic out of the box, treating these
listeners as real entry points, like we do with our Commands, Controllers or
Twig extensions.
