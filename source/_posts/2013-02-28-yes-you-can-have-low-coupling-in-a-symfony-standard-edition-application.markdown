---
layout: post
title: "Yes, you can have low coupling in a Symfony Standard Edition application!"
date: 2013-03-02 19:45
comments: true
categories: [symfony2, php, architecture, solid]
---

Have you ever heard someone say full-stack frameworks are bad because they force you to write high coupling code? Well, that's mostly malarkey! You can fight high coupling just by embracing [S.O.L.I.D][solid] principles and making smart architectural decisions.

<!-- more -->

Please notice:

> #### This article might seem really big, and please excuse me for that, but I really think this topic deserves a lot of thought and attention.
 
And also keep in mind that:

> #### This article is not a how-to type of article or a step-by-step tutorial. Its an article that presents some thougths and conceptual ideas to create better applications using a full-stack framework.

I once asked people on [stackoverflow](http://stackoverflow.com) if [everything should really be a bundle](http://stackoverflow.com/questions/9999433/should-everything-really-be-a-bundle-in-symfony-2) inside of a [Symfony Standard Edition](https://github.com/symfony/symfony-standard) application. That question was actually the trigger for me to start looking deeper into the concepts of the framework and architecture in general.

## Bundles

If you don't know the concept of a bundle, the [documentation](http://symfony.com/doc/current/book/page_creation.html#page-creation-bundles) explains it really well:

> A bundle is simply a structured set of files within a directory that implement a single feature. You might create a BlogBundle, a ForumBundle or a bundle for user management (many of these exist already as open source bundles). Each directory contains everything related to that feature, including PHP files, templates, stylesheets, JavaScripts, tests and anything else. Every aspect of a feature exists in a bundle and every feature lives in a bundle.

The thing is the concept of bundle is actually really misunderstood by some people who use Symfony as a full-stack framework{% fn_ref 1 %}. Without really understanding that concept, people tend to create bundles with lots of dependency between them and high coupling code beucase of the structure they come up with.

It makes a lot of sense to create bundles to integrate third-party libraries or solve specific and common problems into a Symfony application. Bundles like [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle), [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle), [KnpSnappyBundle](https://github.com/KnpLabs/KnpSnappyBundle) or [RespectValidationBundle](https://github.com/Respect/ValidationBundle) are great because they serve this purpose.

But when you are developing a web application, the heart of that application — you can call it the domain{% fn_ref 3 %}, the model or whatever else — is almost always independent from the framework you are using. You need your business specific code to be portable to any framework you want to at anytime during the development of that application.

So the question is: does it make sense to have domain, business specific code inside generic bundles? It really depends, but generally no. Business specific code is, basically, spread throughout a [service layer](http://martinfowler.com/eaaCatalog/serviceLayer.html), a [domain model](http://martinfowler.com/eaaCatalog/domainModel.html) and some other [important architectural concepts](http://martinfowler.com/eaaCatalog/index.html). However, if you know some business code should be reused in two different applications of the same domain, maybe that's the perfect time to wrap them in bundles.

Let's take a look at a quick example: a simple web application, something like a website that holds both a blog and a forum for their users. Let's consider, for the sake of the example, that the forum and the blog are accessed by the same users. Those users use the same login and password for both applications. The `User` entity would be shared by the `ForumBundle` and the `BlogBundle`, right?

    src/
    └── Vendor/
        └── Product/
            ├── BlogBundle/
            ├── ForumBundle/
            └── SiteBundle/

Now comes a simple and recurring question: where should you place the `User` entity? Inside the `ForumBundle`? Or the `BlogBundle`? Shold you create a separate bundle to hold all features related to the user entity, like a `UserBundle`? For that problem, people come up with all kinds of solutions.

I've seen bundles created just with the purpose of holding entities, such as `EntityBundle`. I've also seen bundles that are considered to exist on a different layer in comparison to the other ones, so they come up with a `CommonBundle` or a `AppBundle`. Still, in my opinion, the problem persists. What is the problem again? Having to follow a bundle structure for a non-compliant case. Should the `User` entity really be inside a bundle? And that leads us to the directory structure of the project.

## Directory structure

With the beauty of the Composer [autoloader](http://getcomposer.org/doc/01-basic-usage.md#autoloading) — or any [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) compatible autoloader — you don't have to worry about diverting from the Symfony full-stack directory structure. Just create your directories as you see fit, place your classes in there and you are good to go. Let's take the same example from above to put this in pratice. What if we had a structure like this:

    src/
    └── Vendor/
        └── Product/
            └── Bundle
                ├── BlogBundle/
                ├── ForumBundle/
                └── SiteBundle/
            └── Entity
                └── User.php
            ├── Repository
            └── Service

We now have a much more clean and verbose directory structure, that tells us the exact scope of the `User` entity, which is the whole application itself. Even if only one bundle is going to actually use the that entity or any of the code outside of the bundle structure, it's still a good approach to separate them. In fact, this kind of design and architectural decision is what makes it possible to not have high coupling with the Symfony full-stack framework or any other framework.

So that leaves us with some other questions: what kind of code is left for the bundles themselves? How can you identify what code you should place inside bundles and what code you should isolate from them? Well, the answers for those questions are actually not simple. However, we can start by thinking on the application specific stuff.

The first thing that comes to mind is the [controller](http://symfony.com/doc/master/book/controller.html). The controller is nothing else than a piece of code created to solve the *delivery mechanism*{% fn_ref 2 %} problem. Its job is basically to, having been activated by a [front controller](http://martinfowler.com/eaaCatalog/frontController.html), send a response to the client - on a web application, usually as a HTML page or a JSON response. Nothing else. Really, nothing else.

Composed by a really skinny code, you most likely won't be able to reuse the controllers — and probably won't want to as well, at least in most cases. With that in mind, don't worry about them. Just keep them inside your application specific bundles — in our previous example, those would be `BlogBundle`, `ForumBundle`, `SiteBundle` — and you are fine.

The next thing that comes to mind when we think about application specific code is the [view layer](http://symfony.com/doc/2.0/book/templating.html). Just like the controllers, views are inside the delivery mechanism scope. They only exist because of the delivery mechanism of your application. If that is the web, than the views are probably going to be a bunch of [templates](http://symfony.com/doc/2.0/book/templating.html#templates) to be displayed on the user's screen.

Templates are even more specific than controllers. Rarely you will want to reuse them — maybe their concept or structure, but not as they are originally on each page. Since Symfony handles [template inheritance](http://symfony.com/doc/2.0/book/templating.html#template-inheritance-and-layouts) pretty well, keep them inside your bundles too and they are going to be settled.

With all this informatin, let's take a look at the evolution of our blog/forum/site example, starting by the directory structure with some more stuff on it.

    src/
    └── Vendor/
        └── Product/
            └── Bundle
                └── BlogBundle/
                └── ForumBundle/
                └── SiteBundle/
                    └── Controller/
                        └── IndexController.php
                    └── Resources/
                        └── views/
                            └── index.html.twig
                    └── ProductSiteBundle.php
            └── Entity
                └── User.php
            └── Repository
                └── UserRepository.php
            └── Service
                └── UserPasswordRetrievalService.php

Can you see the actual improvement here? All of your business specific code, the actual heart of your application, will be isolated from default structure of the framework. All you have actually following that structure is code that is, arguably, light and disposable, the controllers and the views. The important part, the part that *really* needs to be testable, maintanable and most importantly independent from any framework or library, is isolated in its own layer.

## Doctrine

One of the greatest things about Symfony Standard Edition is that it already comes packed with Doctrine integration, thanks to [DoctrineBundle](https://github.com/doctrine/DoctrineBundle), from the Doctrine [organization](https://github.com/doctrine). This allows you to start developing your application domain layer, entities and repositories just by following some [conventions](http://symfony.com/doc/master/book/doctrine.html).

Doctrine is a really powerful persistance related set of libraries. One of its best features is the use of [annotations](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#introduction-to-docblock-annotations) to parse documentation and transform it into metadata that can be processed later. Using Symfony, it is really common to see entities defined like this:

``` php src/Vendor/Product/Entity/User.php
<?php
namespace Vendor\Product\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="users")
 */
class User
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;

    /**
     * @ORM\Column(type="string")
     */
    protected $username;

    /**
     * @ORM\Column(type="string")
     */
    protected $password;
}
```

While having the flexibility of using annotations, you also contextualize information about the entities and the way they are persisted in the database inside the class definition ifselt. That is really awesome. If you don't use annotations yet, maybe check out the [presentation](http://www.slideshare.net/rdohms/annotations-in-php-confoo-2013) on them by [@rdohms](https://twitter.com/rdohms).

But everything that has advantages also have disadvantages. In my opinion — and this is really, really personal, the collateral damage annotations can cause in your code is high coupling with the library responsible for parsing them. That's why I tend not to use annotations for domain specific code — entities, repositories, services and such, but only for application specific code.

Symfony also uses the power of annotations to allow you to define [routes](http://symfony.com/doc/2.1/bundles/SensioFrameworkExtraBundle/annotations/routing.html) and [templates](http://symfony.com/doc/2.0/bundles/SensioFrameworkExtraBundle/annotations/view.html), with the power of [SensioFrameworkExtraBundle](https://github.com/sensio/SensioFrameworkExtraBundle) — also packed with Symfony Standard Edition. I use them like there's no tomorrow to define routes and templates for my controllers without really worrying about coupling.

My advice to you, with all that in mind, is to try and decouple your domain specific code the most you can from any library or framework. Using Symfony as a full-stack framework, that means to have all of Doctrine — or any other persistence library — configuration and mapping in isolated files, such as YAML or XML, like:

``` yaml src/Vendor/Product/Resources/config/doctrine/User.orm.yml
Vendor\Product\Entity\User:
    type: entity
    table: users
    id:
        id:
            type: integer
            generator: { strategy: AUTO }
    fields:
        username:
            type: string
        password:
            type: string
```

Yes, that will make you write a considerable amount of aditional code and mantain a larger number of files, but I usually don't worry about it too much. Since you are not going to be following the default Symfony structure, you might have to check the [documentation](http://symfony.com/doc/master/reference/configuration/doctrine.html#mapping-configuration) on the Doctrine mapping configuration and see if you need any additional work to make everything flow as expected.

## Final thoughts

To be honest, if you are comfortable with writting an application using some components to solve individual and isolated problems and not a full-stack framework, you will be able to write low coupling code easier. That doesn't mean every application based on this kind of framework has high coupling code. 

The bottom line is: low coupling it's really more about how you design and structure your application architecture and less to what framework you choose to use. Symfony is extremely powerful and, at the same time, flexible enough to let you decide how to structure your code. That's awesome, right?

Like I said at the beginning of this article, my intent was not to create a step-by-step guide or a tutorial on how to write low coupling code. That's really something that comes with time and experience. My goal was to help you aim to the right direction on how to learn that faster and in a simpler way. And I hope I achieved that.


{% footnotes %}
    {% fn Read it as a Symfony Standard Edition application. %}
    {% fn Concept extracted from a Keynote from Uncle Bob (Architecture, the Lost Years). %}
    {% fn Concept extracted from Erich Evans' Domain Driven Design: Tackling Complexity in the Hearf of Software book %}
{% endfootnotes%}

[solid]: http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)