---
layout: post
category : projects
title: "API first CMS"
tags : [api, cms, nodes, reactjs, swagger]
---
{% include JB/setup %}

For the last few years I've spent a lot of time at Spindrift helping develop Site Builder, which is essentially a high level CMS built into the Oracle ATG BCC. Working on a monolithic stack that provides business configurable content like this has led me to think about how the platform dependent solution could be genericised into working with any modern web application. The simple solution is to create a REST interface that makes the existing definition of the content available for a web application to consume; which is an acceptable solution if you're using Oracle ATG and Spindrift Site Builder or a similar platform specific solution. The more interesting solution is one that's platform agnostic and relies on proper REST APIs to consume, manage and supply content. In other words an API first CMS.

By this I'm not thinking of your typical CMS where you upload assets and spent a lot of time in a WYSIWYG editor creating content, there are plenty of solutions out there that already do a fine job of this. Instead, developers define a template that the web application knows how to render, then business users fill out content using these templates, often pulling content from configured end points (including your typical CMS), and combine the results into a document of renderable content. This document definition is then made available to the web application via an API and rendered appropriately.

![API First CMS Architecture](/assets/posts/API CMS.png "API First CMS Architecture")


The endpoints that content is pulled from could include: other CMS systems, a PIM for the product catalog, an media management system such as Scene7 or any other content provider. The source really doesn't matter as long as it provides an API that provides content in a generic fashion that we can consume.

What we provide are two APIs, one that is used by developers to define the type of content that is available and by business users to create content; the second by the web application to retrieve the content that should be rendered for a request.

As described this should provide us with a nicely decoupled solution that can consume content from any system and allow it to be configured into a JSON definition that can be rendered by the web application.

For the first phase I built out a simple PoC using Swagger to define the management API and, ReactJS to build out the Document Editor application and a simple web application to demonstrate rendering the content. The document, layout and content definitions are defined using JSON Schema. I dummied up the management API using swagger-node and simply stored the resulting data in memory rather than persisting to a NoSQL database, though that will be the eventual location.

![Application Screenshot](/assets/posts/cms_application.png "CMS Poc Application Screenshot")

At this point I have a Document Editor application that allows content to be created and edited, including a crude embedded preview of the document rendered by the web application. This is currently set up to all run on a single EC2 instance but the plan is to build this out on AWS by Dockerising the two web applications and using:
* CloudFormation, to define the resources
* VPC, the Document Editor and management API will live in a different VPC to the Content API and web application, allowing access to be restricted.
* API Gateway, the management API and Content API will be provided using AWS API Gateway.
* DynamoDB, the data will be persisted using DynamoDB tables.
* Lambda, the plan is to use Lambda with the API Gateway to persist and retrieve the content.
* EC2 Container Service, the web application Docker images will likely be managed here.

There is a lot of required functionality that I'm not planning to build out here but that should be kept in mind including:
* Security - I'm making no attempt to lock down access to the Document Editor or Management API besides potentially putting them in their own VPC.
* Versioning - required to support multiple users editing content
* Projects - required to support grouping content changes
* Sites - support for delivering different content for different sites and/or applications
* ACL - application and item based
* I18n of application
* Support for i18n of content
* Personalisation of content - may not require direct implementation, potentially just config for content sourced from external API
