---
layout: post
category : cloud-commerce
title: "Cloud Commerce"
tags : [commerce, cloud, aws, apigateway, cloudformation]
---
{% include JB/setup %}

This is a summary of a presentation I put together on how you might go about transitioning a typical enterprise retailer still running their retail site on monolithic application such as Oracle ATG onto a more modern platform.

## Target
* Large Enterprises that are running archaic monolithic stacks such as ATG
* Enterprises that want to retain control and access to their own stack

## Advantages
* Migration path for existing Commerce stacks to Cloud native solution
* Ability to run entire Commerce Platform in the Cloud
* Modern ways of working: TDD, CI/CD
* Cloud Agnostic
* Vendor Agnostic

## How
Use an API Gateway to provide access to:
* Serverless Functions
* Container based Microservices
* Proxy to third party services
* Proxy to existing monolithic applications

Ideally you don't want to get caught up in trying to host and scale a monolithic application such as ATG in the cloud.

## API Gateway
The API Gateway provides access to number of logical services.

![API Gateway](/assets/posts/cloud-commerce/api-gateway.png)

## Microservices
These Micoservices could be containers or serverless functions, proxies to third party services or proxies to existing monolithic applications.

![Microservices](/assets/posts/cloud-commerce/microservices.png)

## Migration Path
* First implementation would mostly be proxy to existing monolithic application such as ATG
* Eventually the endpoints would be replaced with custom microservices and third party services
* Long term goal would be a custom microservice or third party service for every endpoint
* Going forwards it mean that changes in functionality are as simple as changing the service behind an endpoint

![Migration Path](/assets/posts/cloud-commerce/migration-path.png)

## Products vs Services
If done properly there's an opportunity to build and sell custom services that can be used by other retailers.

There are a couple of ways this can be done and it's debatable what the best solution would be:
* Sell subscriptions to services: overhead of client tracking/billing, requires 24/7 support, potentially provides access to useful data (privacy concern)
* Sell product licenses: harder to enforce licensed use, potential to modify and customise service for client specific requirements

It's likely a combination of the two could be used where some solutions are sold as a service that customers subscribe to, while for others a license is sold and the microservice is deployed into the customers environment.

For some customers the ability to deploy a product into their own environment rather than rely on subscription service may be a big selling point.

## Open Source
* A way to gain traction and mind share; creates potential to sell other products and services that aren't open sourced.
* Benefit from development by other users.

## Tooling
* CI/CD, something to refine and work out during product development
* Would make the roll out of new services and updates to existing services trivial
* Should be applicable to any products or services that are used

## Machine Learning
* Using an API Gateway to decouple applications from the services providing the functionality allows you to role out implementations using ML for any endpoint.
* Having all calls go through the API Gateway not only means that you can easily switch the service provider, it should allow you to record a rich set of data for ML purposes.

## What to build?
* [Cloud Template Generator](http://blog.maxieduncan.co.nz/projects/2017/07/27/commerce-wizard): SPA to generate Cloud Templates for AWS, likely GCP, maybe others?
* [Layout Engine](http://blog.maxieduncan.co.nz/cloud-commerce/2017/11/04/layout-engine): Helps decouple the frontend applications from the backend services while providing the business with control over content and layout
* Frontend application: Decoupled frontend application that makes use of the API provided by the API Gateway
* Site management: Application to manage "site" configuration, an important concept to build in early on
* Common Commerce API: It may not be practical but, having a common interface that you can switch out the services provided for would be powerful
* Proxy Services: Build out Cloud Templates to proxy to existing services: ATG 11.3, OCC, Hybris, Demandware, Shopify, Salesforce, etc
* Custom microservices: Where you want to get to. Having a Common Commerce API would make this easier though isn’t necessary. Potential to explore ML driven content. Management applications to go with them? Management hub? Smart pricing, Smart Recommendations
* AWS Cloud Templates: Consider building Templates to enable hosting existing services in the cloud.
	* Instance based is most straightforward but least desirable, ideally proxy to existing stack instead
	* Mix of containers and instances
	* Containers and cloud services, eg RDS for Oracle, EFS for publishing files, Elastic Search?

## Cloud Template Generator
* SPA to generate Cloud Templates for AWS, GCP, etc
* Potential to open source application
* Open source library of common templates; potentially include our own products and services
* Build a private library of more client specific templates?

## Layout Engine
* Could be offered as a product or a service
* Minimal product is API only, i.e. no management application
* Potential to open source core product (API, Lambda functions); keep other functionality such as the layout management application, url management, form management, e.t.c closed source.
* These applications can be built out over time, including adding features for versioning/workspaces, ACL, Preview, e.t.c
* Build in Analytics recording
* Build service for Multi-Variant testing
* Build service for ML driven content using analytics
* Build in reports based on analytics, own and potentially third party
* Strictly speaking the Layout Engine is entirely optional, however it's something where I can see us being able to add a lot of demonstrable value

## Where to start?
* DevOps tooling: should come about from developing the applications.
* Cloud Template Generator: allows pulling together preconfigured Cloud Templates to easily demonstrate rolling out a customised cloud stack.
* Frontend Application: simple React application to demonstrate functionality, uses minimal Layout Engine to drive content. Demonstrates DevOps tooling to role out application resources.
* Layout Engine: defines the content to render on the site, provides business configurability. If developed as a product would demonstrate DevOps tooling to role out serverless functions, containers and other cloud resources.
* Proxy to existing monolithic API: provide access to existing commerce functionality, potential to work on Common Commerce API, potential for other commerce providers, potential to host instead of proxy.
* ML Services: Pricing? Recommendations? Requires access to data to develop and demo.
* Other?

## Talking Points
* Cloud Hosting
	* Ideally want to retain control of cloud infrastructure
	* Providing our products as subscription model services versus licensing as products
* Ideally you want to only provide:
	* proxies to existing products
	* containers
	* serverless functions
* Ideally you don't want to get into:
	* supplying tars
	* managing instances
* Retaining IP:
	* new services created for custom needs,
	* third party integrations
* Retaining data, especially in regards to future ML purposes
