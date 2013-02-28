---
layout: post
title: "Yes, you can have low coupling in a Symfony Standard Edition application!"
date: 2013-02-28 09:45
comments: true
categories: [symfony2, php, architecture, solid]
---

Have you ever heard someone say full-stack frameworks are bad because they force you to write high coupling code? Well, that's mostly malarkey! You can fight high coupling just by embracing SOLID principles and making smart architectural decisions.

<!-- more -->

I once asked people on [stackoverflow](http://stackoverflow.com) if [everything should really be a bundle](http://stackoverflow.com/questions/9999433/should-everything-really-be-a-bundle-in-symfony-2) inside of a Symfony Standard Edition application. That question was actually the trigger for me to start looking deeper into the concepts of the framework and architecture in general.

## Bundles

If you don't know the concept of a bundle, the [documentation](http://symfony.com/doc/current/book/page_creation.html#page-creation-bundles) explains it really well:

> A bundle is simply a structured set of files within a directory that implement a single feature. You might create a BlogBundle, a ForumBundle or a bundle for user management (many of these exist already as open source bundles). Each directory contains everything related to that feature, including PHP files, templates, stylesheets, JavaScripts, tests and anything else. Every aspect of a feature exists in a bundle and every feature lives in a bundle.

The thing is the concept of bundle is actually really misunderstood by the people who use Symfony as a full-stack framework{% fn_ref 1 %}. Without really understanding that concept, people tend to create bundles with lots of dependency between them and high coupling code beucase of the structure they come up with.

It makes a lot of sense to create bundles to integrate third-party libraries into a Symfony full-stack application. Bundles like [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle), [FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle), [KnpSnappyBundle](https://github.com/KnpLabs/KnpSnappyBundle) or [RespectValidationBundle](https://github.com/Respect/ValidationBundle) are great because they serve this specific purpose.

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

{% footnotes %}
    {% fn Read it as a Symfony Standard Edition application.  %}
{% endfootnotes%}