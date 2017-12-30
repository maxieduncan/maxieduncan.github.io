---
layout: post
category : projects
title: "Commerce Wizard"
tags : [commerce, cloud, aws, apigateway, cloudformation]
---
{% include JB/setup %}

## Commerce on AWS

I've been thinking recently about how you’d put together a Commerce application on AWS and it’s taken me down an interesting avenue. I haven’t looked deeply into what is already out there in this space but the high level aim here would be:
* API driven frontend applications
* make use of existing products and AWS services, giving a choice of which are used
* ability to provide custom services
* use AWS Cloud Formation to quickly role out a new stack

This led me to put together this basic diagram:
![Cloud Commerce Architecture.png](/assets/posts/Cloud-Commerce-Architecture.png)

The thought here was that you’d have different Cloud Formation templates ready to role out based on the products/services being used, using API Gateway to provide a single access point for the front end applications.

## Commerce Wizard
What I’ve started thinking is that it would actually be quite useful to a have a wizard that lets you select the products/services (defined in a JSON configuration file) that you want to use and then generates a Cloud Formation template that can be used to launch a full commerce stack.

![Commerce Wizard Steps.png](/assets/posts/Commerce-Wizard-Steps.png)

I see there essentially being four major steps to this wizard:

### 1) Cloud Provider
Starting out with AWS, if designed properly there isn't any reason you wouldn't be able to support GCP or others as the primary platform.

### 2) Service Providers
This would mean selecting a list of service providers that provide the actual functionality of the site, then optionally the endpoints they make available. In some cases these services would require deploying assets into AWS, in others it could just be configuring access to existing APIs.

You’d have the option to configure custom endpoints of course and we’d be able provide our own services.

Once you’ve selected all the services and endpoints you should be able to provide a preview of the API that will be provided using Swagger or similar.

Ideally you want a common API but this isn’t always practical or even ideal. Using the API Gateway would allow you to put in place logic to provide a common API from various services but as you’ll see I’m not sure it’s strictly necessary.

Potential Services:
* Commerce
* Catalog/PIM
* Search
* Payment Gateway
* Pricing
* Promotions
* CRM
* Site
* Analytics
* Customer Service
* Address lookup
* Images
* Video
* Others ...

Potential Service Providers:
* <Custom> (Proxy) - proxy to services sold by us
* <Custom> (AWS) - deploying our own services into AWS
* Customer (Proxy) - proxy to customers existing services
* Customer (AWS) - deploying customer services into AWS
* AWS - access to existing AWS services, e.g. Elastic Search, Cognito, etc
* AWS Lambda - delegate to AWS Lambda function
* Third Party Service (Proxy) - proxy to third party service/product
	* Apigee Open Commerce - proxy to third party implementation
	* ATG 11.3 (Proxy) - proxy to existing ATG system
	* OCC - proxy to OCC
	* Any other service offering a useful API ...
* Third Party Service (AWS) - deploy third party service into AWS
	* Apigee Open Commerce - deploy third party implementation
	* ATG 11.3 (AWS) - deploy ATG stack into AWS
	* Any other service offering a useful API that could be deployed ...

### 3) Layout Engine
Optionally you could also select a “Layout Engine”. This is what I had in mind when doing the [API First CMS PoC](http://blog.maxieduncan.co.nz/projects/2017/03/27/api-first-cms) at the start of the year and I still see some value in this. This is very much along the lines of what Site Builder and OCC offer but I see the opportunity to offer a more flexible offering.

The idea here would be you could offer one or more of these business configurable Layout Engines, then based on the service providers and endpoints selected, populate it with preconfigured templates. For example you could imagine templates provided to: pull content from a CMS, create a gallery sourced from a particular endpoint, manage the cart and profile, etc based on the providers selected.

The advantage of this is being able to immediately deploy a business configurable website that is ready to use with the selected services. If the templates are linked to particular service providers and endpoints that they support, then the need for a common API becomes less necessary as you only deploy the supported templates.

There's potentially other services to be build around this such as URL management, site map generators, etc.

### 4) Reference Application
Optionally you could also generate a reference application. This could populate a git repository with a sample front end application based on the services and endpoints that have been selected and deploy it using ECS.

In the case of a Layout Engine being used this would mean an application that knows how to render the templates that have been deployed in the prior step and optionally sample data; if a Layout Engine isn’t being used this could be a sample reference application that uses the services and endpoints selected.

The benefit here is twofold, a starting point for customers own implementations and a convincing demo.

## Cloud Formation
The outcome of all this would be a Cloud Formation template that would deploy your API Gateway, Lambda Functions, EC2 instances, ECS, VPCs, etc based on the configuration you selected.

It should be possible to store the wizard configuration as JSON and allow it to be reloaded and modified in the future as well, potentially adding new services or changing providers. Maybe this could even be done by loading a previously created Cloud Formation template.

There is potential to easily swap services if there’s a common API in place but even without one, if you have a layout engine, you should be able to role out new templates and components for new endpoints easily.

## OCC
It's been interesting getting a glimpse at OCC and it actually seems like a decent enough offering in many ways but with some major limitations (at least currently).

They've taken a very restrictive black box approach in their implementation and it looks very much as though it's backed by tightly coupled monolithic services. This limits them quite significantly as far as customisation and extendability are concerned, something I think will only be partially alleviated by the server side nodejs extensions they're starting to enable.

Their version of a "Layout Engine" is similarly restricted, being restrained to Bootstrap and Knockout JS and the data model being explicitly linked to the views.

##  Summing Up
I think that this would be targeted at retailers that have existing service providers but are looking to move to a more modern architecture and keep flexibility around their suppliers. You could imagine being able to deploy a cloud based, API driven React application for example that has been set up to connect to ATG's Commerce API. That ATG stack could either be in their own data centre or potentially you could even provide a Cloud Formation template to run it in AWS as part of the wizard. Of course ideally ATG isn't involved at all.

There are a lot questions around how this could work, some of the bigger ones:
* Is there value in this for anyone?
* How do you monetise it?
* Would we want to license and deploy our own services into customer stacks or provide them as services only accessible via API and bill for usage?
* Do you want to handle deploying and running the entire stack and dealing with support? If you do you could potentially target new and smaller retailers.
* Or provide them with just the template and offer consulting services and/or provide commerce services and charge for those?
* How would interaction between the different services work in the back end?
* Does trying to offer this much flexibility make it too complex to implement?

What are the benefits:
* No vendor lock in, ability to easily switch out suppliers if they provide the same/similar APIs. Different data payloads and definitions could complicate
* Variety of service providers ready to go, easy to add new ones
* For enterprises, ability to retain ownership of the entire stack
* Business configurable site layout
* Choice of technology for front end applications
* Reusable Cloud Formation templates

Things to consider:
* Security
* Logging (especially across multiple service providers)
* Reporting
* Monitoring
* Compliance (handily API Gateway and Lambda functions are [PCI Compliant](https://aws.amazon.com/compliance/services-in-scope/), giving some options)
* Common API, this has the potential to ease swapping services in and out very easily
* CI/CD, for services provided or even just set up for the front-end application (s)
* Having the deployment template should make it easy to deploy environments for testing and development
* Transfer of data between environments
* Complexity added by having numerous products and services, especially in regards to data and different environments
* Support for other API Gateways like Apigee
* The wizard is only useful if there are services being deployed to the cloud, if you're just proxying to existing services then something like Apigee is probably more appropriate
* The concept of projects/workspaces for merchandisers, unlikely to be possible across different products and services. Primarily an issue for the Layout Engine.

What else could be built out to support this:
* Layout Engine, optionally including:
	* Layout Management Application, for managing template definitions rather than managing via an API
	* URL Management
* Reference Application(s)
* Custom Services
	* Smart Pricing Engine
	* Commerce Engine
	* ...
* Site Management Application
* Back end integrations between different service providers
* Commerce Management Application, for managing configuration
* Providing a common access point for these applications

## Appendix A
What the Commerce Wizard could look like:

![Commerce Wizard Mock](/assets/posts/Commerce-Wizard-Mock.png)

## Appendix B
What the JSON configuration for the Commerce Wizard could look like:

```{
	"version" : 1,
	"owner" : "<Custom>",
	"providers" : {
		"apigeeCommerceStartingPoint" : {
			"name" : "Apigee Open Commerce <Custom>",
			"template" : <Provider Specific Template>
		},
		"apigeeCommerceProxy" : {
			"name" : "Apigee Open Commerce (Proxy)",
			"template" : <Provider Specific Template>
		},
		"oracleAtg11_3" : {
			"name" : "Oracle ATG 11.3 (AWS)",
			"template" : <Provider Specific Template>
		},
		"oracleAtg11_3Proxy" : {
			"name" : "Oracle ATG 11.3 (Proxy)",
			"template" : <Provider Specific Template>
		},
		...
	},
	"services" : {
		"commerce" : {
			"name" : "Commerce"
			"providers" : {
				"apigeeCommerceStartingPoint" : {
					"name" : "Apigee Open Commerce <Custom>",
					"template" : <Service Specific Template>,
					"endpoints" : [
						{
							"url" : "/cart",
							"method": "POST",
							"description": "Create a new Cart",
							"template" : <Endpoint Specific Template>
						},
						...
					]
				}
			}
		},
		...
	},
	"layoutEngines" : {
		"startingPoint" : {
			"name" : "<Custom>",
			"template" : <Layout Engine Template>,
			"layoutEngineTemplates" : {...},
			"referenceApplications" : [
				{
					"name" : "React",
					"template" : <Layout Engine App Template>,
					"path" : "<path to app repository/image>",
					"data" : {...}
				},
				...
			]
		},
		...
	},
	"referenceApplications" : [
		{
			"name" : "React",
			"template" : <Reference App Template>,
			"path" : "<path to app repository/image>"
		},
		...
	]
}```

This provides the option to configure Cloud Formation templates at the provider, service and endpoint levels as well as for the Layout Engines and Reference Apps.

You should be able to process the Cloud Formation templates to determine which AWS services are being used rather than having to explicitly define them.
