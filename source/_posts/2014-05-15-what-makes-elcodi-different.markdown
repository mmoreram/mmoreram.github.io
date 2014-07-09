---
layout: post
title: "What makes Elcodi different?"
date: 2014-05-15 17:40
comments: true
author: Marc Morera
categories:
    - Elcodi
    - Ecommerce
---

Well, the league of web development, in which we play, is a fussy league, I 
started to recieve the first questions about the project I find myself 
developing... ¿ Why a new project called Elcodi wants to cover the needs of 
ecommerce when there is something similar called Sylius?

Although I twitt that I would write a comparison between both platforms, I 
firmly believe that it is absurd to compare them, since Sylius is a 
very mature project and has lots of features we do not have ( nor will we have 
short-term ). So, this post is nothing more than a declaration of intentions 
about why we have decided to start a separated project from the existing 
options.

Firstly, I want to express complete respect to Sylius. It's a project 
built by many different contributors, and I do not want to compare their 
results with our ambitions at all. I think they both are projects with 
different perspectives, and since one of our most important value ​​is humility, 
we start as a small project. Ambitious, but for now, small.

Secondly, I'd like to share with the community what I think are the three basic 
pillars in our project. All three are equally important and we strongly believe 
that one feeds the others. We are very strict on these three aspects, and it 
will remain as it is throughout the years.

Event-Driven Design
-------------------

Elcodi is a completely event-oriented project. All components have a set of 
events, especially designed to customize the behavior of a specific action.

Not giving importance to this point means that many people will have to fork 
the project itself to implement their own behaviors, and this is not a good 
philosophy. A third party component should behave as such, giving a high degree 
of customization.

Clean Code
----------

Our first and most important objective is to provide developers with a set of 
useful tools for working their own implementations. We do not claim to be the 
all-purpose and immediate solution for existing projects. Our desire is to 
provide a completely customizable workbench in order to solve those recurring 
ecommerce projects problems.

For example, we can not deliver an all-inclusive Cart management solution for obvious reasons: each 
project has its own unique business needs. What we can do is to propose a 
simple common behavior, thus giving the opportunity to evolve by offering a 
completely customizable model, giving the opportunity to extend behaviors thanks
to the proposed event structure and by offering a decent and growing documentation.

In my view, realistically, since real needs generate from a heterogeneous community 
composed by people with different levels of skills and experience, we will just 
collect these needs. Consequently, we will take care of the programming style, 
clean code design and standardization so that every line of code can be understood 
by everyone. 
We are not die hard fans or disciples of any specific software development process
or methodology, we won't be relentlessy chasing latest trends just for the sake of it. 
We just want to create a valuable project, both for developers who 
want something nifty and stable for small quick project and for those who believe 
that their e-commerce will grow much. 

Working on an Elcodi based project must be fun and healthy.


Business perspective
--------------------

We are a technology project, but first and foremost we are a company. This means 
that though we aspire to be fully open source, we do have business model 
behind, defined by a financial structure, which is based on constant growing.

As you can see on our website, our key role in the process is the Market. As said, 
one of the great advantages that event-driven architecture gives us is that it can 
easily be extended: third party modules will be simple to integrate. 

Moreover, a professional team of developers will be responsible for the project, 
to train interested people and review the quality of the extensions submitted 
to the market. 

At the same time, developers who wants to base their implementations on Elcodi 
will likely find it easy to cover their needs, through open-source modules, market
extensions or by coding the solutions themselves (and of course selling it in the
Market if it complies with Elcodi code rules!)  

Let me call this escenarios a win-win.

Final thoughts
--------------

I repeat myself. We do not want to compete with anyone. I understand that every 
project has its trajectory. What appears in this post is our. Cross-project
collaboration is  essential, and we will commit to work alongside other projects, 
because at last, everyone gains :)

Soon we will publish a roadmap, as well as a set of training modules 
for people who want to learn some cookie recipes of web architecture and a 
little more about our project.

If you want to know about project updates, follow our activities in 
[Twitter - @elcodi_dev](http://twitter.com/elcodi_dev) or follow Elcodi 
development in [Github - @elcodi](http://github.com/elcodi/elcodi). 

Obviously we want people to clone the project, inspect it, question us, discuss 
constructively and give us reviews on our implementations. Contributions to the 
project will be well received and appreciated :)