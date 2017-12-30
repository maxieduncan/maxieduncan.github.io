---
layout: post
category : cloudcommerce
title: "Commerce Wizard PoC"
tags : [commerce, cloud, poc, angular2, aws, apigateway, cloudformation]
---
{% include JB/setup %}

I've had a bit of a play at building my [Commerce Wizard](http://blog.maxieduncan.co.nz/projects/2017/07/27/commerce-wizard) creating a simple [Proof of Concept](http://commerce-wizard.s3-website-eu-west-1.amazonaws.com).  It's functional enough as a PoC and you get the idea but there's plenty more that could be added and refined; I've also only put together a minimal set of data to demo it. Chose Angular2 as the framework because it seemed like a good opportunity to learn something new and I've got to know CloudFormation a bit better.

It certainly seems like a viable idea though the benefits of the actual application are pretty limited. Any real advantage would be gained by building up a library of reusable CloudFormation templates rather than in the actual wizard itself, though it certainly does make them easier to combine together.

As with most cloud services, encountered some limitations worth noting along the way (and I'm only scraping the surface of what's available):
* limit of 60 input parameters per template, not necessarily an issue (and certainly not in the existing example) but if you're§ trying to combine a range of disparate services it could require the template to be broken up into pieces (probably linked together by CloudFormation outputs).
* API Gateway swagger definitions need to inline for the CloudFormation parameters and constants to be resolved. If you need access to these variables and attributes then referring to a swagger definition stored in an S3 bucket won't work (took a while to work that one out).

You'd need to put some thought into how you'd handle different environments. As an example, my [API First CMS](http://blog.maxieduncan.co.nz/projects/2017/03/27/api-first-cms) currently has the DynamoDB table names hardcoded in the Lambda functions. If you want to create distinct environments - as you would - then some effort would need to go into the CloudFormation templates to ensure that they can be uniquely identified.

There's also a question over how you'd handle data that I haven't really looked into. It depends on the context of course, but I'd expect you'd probably copy any relevant data to a bucket and then as outputs list the commands that need to be run to import that data, or maybe just the file locations. Potentially the execution of the CloudFormation template and the loading any data could even be run directly from the wizard.

## Usage
The obvious application of the wizard would be to make a nice starting point for new projects and for demos. Assuming you have a broad library of templates for different services to choose from, it should greatly simplify generating a CloudFormation template allowing you to quickly get started.

It's also an interesting exercise to think how this could be used to help a CI/CD pipeline. Ideally you want to be able to easily set up isolated environments for development and testing that use minimal resources and be able to automatically push you're changes to any services all the way through to production.

### Environments
There's potential to have a variety of templates to create temporary stacks for development and testing purposes, in addition to the more regular stacks for Production, Test and Development environments.

[image:./A80AA4FD-CB4E-42F0-B169-A20DBD37F4BD.png]

#### Shared Services
For development and testing, to reduce the number of cloud resources being created it would make sense to have shared stack/s that can be proxied to, rather than duplicating all the services and their end points for each stack that is created.

This leads conceptually to several types of CloudFormation templates:
*Isolated*: Deploys the assets just for the service being worked on
*Frontend*: Deploys just a frontend instance, plus API Gateway that proxies to shared environment
*Integration*: Deploys the assets just for the service being worked on, plus API Gateway that proxies to shared environment
*Mixed*: Deploys the assets just selected services, plus API Gateway that proxies to shared environment
*Full*: Deploys the assets for every service being worked on

#### Service Sources
There are probably four types of sources you'd expect for the API Gateway:
* Proxy
* Serverless (Lambda, Functions)
* Container
* Instance
With proxies you can't do much more than change the configured endpoints, with functions and containers you can deploy assets that you've generated, instances are probably a little more complex and it's questionable in this context why you'd choose to use them over containers.

#### Developing
Starting with development, you can imagine generating a variety of CloudFormation templates to create different developer stacks depending on what's being worked on. When working on a specific service, a developer selects the appropriate CloudFormation template (or creates a new one if necessary), deploys a new stack and gets to work.

The different templates should give a huge amount of flexibility in the number of resources actually deployed, while providing developers a sandbox to develop the services they're working on without impacting others.

#### Testing
It's all about CI/CD and automation. At some point this would likely involve deploying customised stacks using CloudFormation templates. You can imagine deploying different stacks to test services in isolation or as part of an integration test with other services.

Dealing with dependencies here could be interesting. You'd hope that dependencies would have been developed and deployed already, but it's likely that in some cases there would be circular dependencies for services that have been developed at the same time and rely on each other. If that's the case some customisation would be necessary; I can imagine a set of parameters being passed into a CloudFormation template specifying the dependencies to use.

## Final Thoughts
Building out the wizard was a fun little project but like I said any real benefit comes from creating a library of reusable templates.

I think there's some real potential here to make things easier but a lot of that would depend on how development and testing is done and how your CI/CD pipeline works. It's something to keep in mind if we ever get to the point where we look into these processes though.
