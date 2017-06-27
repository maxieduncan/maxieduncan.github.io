---
layout: post
category : projects
title: "API first CMS: AWS API Gateway, Lambda & DynamoDB"
tags : [api, apigateway, aws, cms, dynamodb, lambda, nodejs, REST, swagger]
---
{% include JB/setup %}

# AWS Integration

This is the part of the project that I've been looking forward to the most, getting rid of the local swagger-node instance and replacing it with AWS API Gateway and persisting the data to DynamoDB using Lambda functions.

### API Gateway

As I already had a Swagger definition of my REST API it was simply a matter of uploading it into API Gateway then configuring the endpoints to use my new Lambda functions. I chose the proxy option, where the Lambda functions are responsible for the status code, headers and body content returned.

The biggest complication in this aspect was the CORS configuration which you have to set up individually for each end point. In this case I wanted all the GET requests to support requests from any source (as both my Document Editor and preview applications require access and the options are either: none, all or a single specific host), and a subset of the POST and PUT requests (the ones for creating and updating the Document and Content items). The GET requests simply required adding the "Access-Control-Allow-Origin" to the headers returned by my Lambda functions; the PUT and POST requests required the addition of OPTION endpoints. For these I found that the API Gateway did a poor job of generating them for the mock end points, not actually populating the required headers on the end points. In the end I manually created the OPTION end points as Mock responses and added the appropriate headers.

### Lambda

Embracing a fully server less approach for the API I chose Lambda as the method to implement the API functionality. I kept NodeJS as the language as it's well suited to the task in hand of processing JSON payloads. The implementation from the node-swagger stack wasn't of much use except as a reference of the functionality required.

I chose to implement functions for the three main areas in my API: Document, Content and Schema (I also created a fourth for the dummy product content). These functions handle the responses for multiple endpoints in their area of functionality in the API:

Document: Used to create, edit and retrieve the Document items.
Content: Used to create, edit and retrieve the Content items.
Schema: Used to create, edit and retrieve the JSON and UI schemas for both documents and contents.

I encountered some errors in my Lambda functions when run through the API Gateway where I hadn't added enough defensive coding and some of the event parameters (specifically pathParameters) weren't present, creating errors that I had to dig into the CloudWatch logs to find.

### DynamoDB

My original implementation had been designed to be used with a NoSQL database so the DynamoDB implementation was rather easy. I set up a role that gave my Lambda functions access to DynamoDB then used the AWS Dynamo API for NodeJS to create, edit and retrieve content from the tables.

I scripted the creation of the DynamoDB tables and the loading of my demo data into them. For the import AWS have their own JSON format that requires you to specify the data type of each JSON attribute rather than just being able to import the JSON themselves. I didn't have much demo data to convert so I did it manually though there are tools out there that automate the conversion if you do have a lot of data.

### Final Thoughts

I did most of the API Gateway and Lambda development through the AWS Console, this probably would have been easier if I'd developed it locally using something like [localstack](https://github.com/atlassian/localstack) then uploading the finished product into AWS.

The finished implementation is very much AWS specific, converting to another API provider would not be entirely straightforward. While the code in the Lambda functions is reasonably generic the DynamoDB references would need to be changed to another NoSQL provider.

All in all the conversion was a pretty straightforward process, taking only a day. The configuration of the API Gateway was the most tedious exercise due to the CORS configuration and now that I have a template I think any major extensions will be made by modifications to the swagger file rather than relying on interface in the AWS console.

At the end though I now have my two React applications running as Docker images in EC2 Container Service and connecting to the API Gateway where Lambda functions store the CMS content in DynamoDB.
