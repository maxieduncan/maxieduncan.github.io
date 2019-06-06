---
layout: post
category : aws
title: "Infrastructure as Code"
tags : [aws, s3, cloudfront, route53, react, lambda, apigateway, dynamodb, serverless, cloudformation, codebuild, codepipeline]
---
{% include JB/setup %}

> *Infrastructure as code* (*IaC*) is the process of managing and provisioning computer  [data centers](https://en.wikipedia.org/wiki/Data_center)  through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools. [Infrastructure as code - Wikipedia](https://en.wikipedia.org/wiki/Infrastructure_as_code)

Infrastructure as Code is something that I've tried to stick to from the beginning of rolling out the Ecogy Asset Management platform. To that end I've made use of serverless.com to define a large subset of the resources that make up the Ecogy platform but, it wasn't until we recently transitioned to using multiple AWS accounts, that I realised how many of the resources were actually being manually maintained.

The main reason for the introduction of multiple AWS accounts is that it allows us to sandbox the different environments, especially important as we bring on new developers. It also allows us to track our AWS spending much more accurately (production versus development, AMS versus BPM) but more importantly helps us secure the production environments from inadvertent issues created during day to day development.

To this end we've transitioned from running everything in one account to isolating the production/test environments in a more restricted production only account and a more open developer focussed account. The recent addition of a BPM platform to our development account has further been isolated in its own account.

![AWS Accounts](/assets/posts/infrastructure/accounts.png)

This has been enabled to a large degree by following the paradigm of "Infrastructure as Code". While this is something that I've followed from the start through the use of `serverless`  and `AWS Cloud Formation` there were a number of short cuts taken at the start that have only recently been rectified to allow us to easily deploy the Ecogy platform to new accounts; in particularly the creation of the CodePipeline and CodeBuild definitions that make up our CI/CD process.

![Infrastructure as Code](/assets/posts/infrastructure/code.png)

The orange areas highlight the infrastructure that is managed by code (with the dotted areas highlighting the areas that still need to be migrated). With the vast majority of the infrastructure now managed by code (or soon to be) and scripted deployments now available we're in a position to role out new sandboxed environments (either by creating new accounts, or deploying a new stage in an existing account).

There are some exceptions to this, the primary one being Cognito User Pools. We have a number of custom properties on the users and changing these is a destructive operation in cloud formation. What this means is that if we added a new custom property to our Cognito users, the cloud formation update would delete the entire User Pool, including all our user accounts, before recreating it with the new property. Not a great scenario for us so these are manually managed. We could maintain a template that allows us auto create the User Pool for NEW environments of course but that's not a high priority at this point and all existing User Pools would still need to be manually updated.
