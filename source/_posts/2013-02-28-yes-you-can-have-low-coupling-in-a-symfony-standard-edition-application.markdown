---
layout: post
title: "Yes, you can have low coupling in a Symfony Standard Edition application!"
date: 2013-02-28 09:45
comments: true
categories: [symfony2, php, architecture, solid]
---

Have you ever heard someone say full-stack frameworks are bad because they force you to write high coupling code? Well, that's mostly malarkey! You can fight high coupling just by embracing [S.O.L.I.D][solid] principles and making smart architectural decisions.

<!-- more -->

I once asked people on [stackoverflow](http://stackoverflow.com) if [everything should really be a bundle](http://stackoverflow.com/questions/9999433/should-everything-really-be-a-bundle-in-symfony-2) inside of a Symfony Standard Edition application. That question was actually the trigger for me to start looking deeper into the concepts of the framework and architecture in general.

## Bundles

If you don't know the concept of a bundle, the [documentation](http://symfony.com/doc/current/book/page_creation.html#page-creation-bundles) explains it really well:

> A bundle is simply a structured set of files within a directory that implement a single feature. You might create a BlogBundle, a ForumBundle or a bundle for user management (many of these exist already as open source bundles). Each directory contains everything related to that feature, including PHP files, templates, stylesheets, JavaScripts, tests and anything else. Every aspect of a feature exists in a bundle and every feature lives in a bundle.

The thing is the concept of bundle is actually really misunderstood by the people who use Symfony as a full-stack framework{% fn_ref 1 %}. Without really understanding that concept, people tend to create bundles with lots of dependency between them and high coupling code beucase of the structure they come up with.

It makes a lot of sense to create bundles to integrate third-party libraries or solve specific and common problems into a Symfony full-stack application. Bundles like [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle), [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle), [KnpSnappyBundle](https://github.com/KnpLabs/KnpSnappyBundle) or [RespectValidationBundle](https://github.com/Respect/ValidationBundle) are great because they serve this purpose.

But when you are developing a web application, the heart of that application — you can call it the domain, the model or whatever else — is almost always independent from the framework you are using. You need your business specific code to be portable to any framework you want to at anytime during the development of that application.

So the question is: does it make sense to have domain, business specific code inside generic bundles? It really depends, but generally no. Business specific code is, basically, spread throughout a [service layer](http://martinfowler.com/eaaCatalog/serviceLayer.html), a [domain model](http://martinfowler.com/eaaCatalog/domainModel.html) and some other [important architectural concepts](http://martinfowler.com/eaaCatalog/index.html). However, if you know some business code should be reused in two different applications of the same domain, maybe that's the perfect time to wrap them in bundles.

Let's take a look at a quick example: a simple web application, something like a website that holds both a blog and a forum for their users. Let's consider, for the sake of the example, that the forum and the blog are handled by the same users. Those users have access to both applications with the same login and password. The `User` entity would be shared by the `ForumBundle` and the `BlogBundle`, right?

    src/
    └── Vendor/
        └── Product/
            ├── BlogBundle/
            ├── ForumBundle/
            └── SiteBundle/

Now comes a simple and recurring question: where should you place the `User` entity? Inside the `ForumBundle`? Or the `BlogBundle`? Shold you create a separate bundle to hold all features related to the user entity, like a `UserBundle`?

For that problem, people come up with all kinds of solutions. I've seen bundles created just with the purpose of holding entities, such as `EntityBundle`. I've also seen bundles that are considered to exist on a different layer in comparison to the other ones, so they come up with a `CommonBundle` or a `AppBundle`. Still, in my opinion, the problem persists. What is the problem again? Having to follow a bundle structure for a non-compliant case. Should the `User` entity really be inside a bundle?

And that leads us to the project directory structure.

## Directory structure

With the beauty of the Composer [autoloader](http://getcomposer.org/doc/01-basic-usage.md#autoloading) — or any [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) compatible autoloader — you don't have to worry about diverting from the Symfony full-stack directory structure. Just create your directories as you see fit, place your classes in there and you are good to go. Let's take the same example from above to put this in pratice. What if we had a structure like this:

    src/
    └── Vendor/
        └── Product/
            └── Bundle
                ├── ForumBundle/
                └── SiteBundle/
            └── Entity
                └── User.php
            ├── Repository
            └── Service

We now have a much more clean and verbose directory structure, that tells us the exact scope of the `User` entity, which is the whole application itself. Even if only one bundle is going to actually use the that entity or any of the code outside of the bundle structure, it's still a good approach to separate them. In fact, this kind of design and architectural decision is what makes it possible to not have high coupling with the Symfony full-stack framework or any other framework.

So that leaves us with some other questions: what kind of code is left for the bundles themselves? How can you identify what code you should place inside bundles and what code you should isolate from them? Well, the answers for those questions are actually not simple. However, we can start by thinking on the application specific stuff.

The first thing that comes to mind is the [controller](http://symfony.com/doc/master/book/controller.html). The controller is nothing else than a piece of code made to solve the *delivery mechanism*{% fn_ref 2 %} problem. Its job is basically to, having been activated by a [front controller](http://martinfowler.com/eaaCatalog/frontController.html), send a response to the client - on a web application, usually as a HTML page or a JSON response. Nothing else. Really, nothing else.

Composed by a really skinny code, you most likely won't be able to reuse the controllers — and probably won't want to as well, at least in most cases. With that in mind, don't worry about them. Just keep them inside your application specific bundles — in our previous example, those would be `BlogBundle`, `ForumBundle`, `SiteBundle` — and you are fine.

The next thing the comes to mind when we think about application specific code is the [view layer](http://symfony.com/doc/2.0/book/templating.html). Just like the controllers, views are inside the delivery mechanism scope. They only exist because of the delivery mechanism of your application. If that is the web, than the views are probably going to be a bunch of [templates](http://symfony.com/doc/2.0/book/templating.html#templates) to be displayed on the user's screen.

Templates are even more specific than controllers. Rarely you will want to reuse them — maybe their concept or structure, but not them as they are originally on each page. Since Symfony handles [template inheritance](http://symfony.com/doc/2.0/book/templating.html#template-inheritance-and-layouts) pretty well, keep them inside your bundles too and they are going to be settled.

With all this informatin, let's take a look at the evolution of our blog/forum/site example, starting by the directory structure with some more stuff in it.

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

Can you see the actual improvement here? All of your business specific code, the actual heart of your application, will be isolated from default structure of the framework. All you have actually following that structure is code that is, arguably, light and disposable, the controllers and the views. The important part, the part that *really* needs to be testable and maintanable, is apart from that.

## Doctrine



{% footnotes %}
    {% fn Read it as a Symfony Standard Edition application.  %}
    {% fn Concept extracted from a Keynote from Uncle Bob (Architecture, the Lost Years). %}
{% endfootnotes%}

[solid]: http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)