---
layout: post
category : events
title: "AWS Summit New York 2019"
tags : [event, aws, cloud]
---
{% include JB/setup %}

This was my first time attending the AWS Summit in New York, previously I've been UK based and attended the one in London. This was considerable larger and I'd say less personable, not entirely surprising with over 11,000 attendees. The format was very much the same (though the London version is two days rather than one) with a Keynote in the morning followed by a variety of specialised talks where you could choose the ones that interested you to attend.

### Keynote

Werner Vogels was the key note speaker and started with his usual exuberance, something that was tempered for the first 15 minutes or so by a number of protestors interrupting him about Amazons relationship with providing services to ICE. It's a somewhat strange choice of venue for protesting, disrupting a technical conference with no political agenda to complain about their own government. What friends they may have had in the audience were mostly turned off before the last protestor was escorted out.

The key note was pretty standard fair, reiterating a number of the concepts that make AWS work so well and focussing on security and how it's built into every AWS service. There were some announcements on new products but most of these are saved for re:invent at the end of the year:
* AWS event bridge
* Visual Studio Code plugin generally available
* AWS CDK generally available
* SageMaker managed spot training
* https://aws.amazon.com/blogs/aws/aws-new-york-summit-2019-summary-of-launches-announcements/

I attended four sessions before the jet lag kicked in and I decided to beat the rush and skip the last session.

### Logging

This talk was focussed on the [Amazon Elasticsearch Service](https://aws.amazon.com/elasticsearch-service/) and using it to process your logs.
[ELK](https://aws.amazon.com/elasticsearch-service/the-elk-stack/): Elastic search, Logstash, Kibana.
Amazon Elasticsearch Service, E&K.

One of the core concepts seemed to be getting the log data into a cheap persistent source such as S3, then loading the data you need into Amazon Elasticsearch Service. With S3 this would typically be done with events and lambda functions to load the latest data into Amazon Elasticsearch Service, with older data being configured to expire as it becomes less relevant for search (as it's stored in S3 it can always be loaded on demand at a later date if necessary).

There are services such as CloudWatch that can be connected directly to Amazon Elasticsearch Service.

### 12 factor app serverless

The [twelve factor app](https://12factor.net) is a popular methodology for programming and the focus on this talk was how it could be applied to serverless programming.

The nature of serverless means that a number of the 12 factors aren't actually relevant, as they're built in to the nature of serverless or simply aren't applicable.

From my perspective it sounded like we are already following all the best processes that were recommended. The one thing I noted was to look into using X-ray tracing for lambda function performance, as it's not something we are currently utilising.

### CDK

The announcement of the general release of CDK 1.0 was made at the key note. This talk was a dive into how it works with something useful examples used to walkthrough the tooling.

We currently use [serverless](serverless.com) and there was nothing in this talk to make me want to change that. Having said that, AWS CDK is programatic (more like terraform) with a selection of programming languages available, whereas serverless is entirely configuration based. This gives you some interesting possibilities to add logic and loops to your stack definition that serverless and CloudFormation (which serverless, AWS CDK and terraform all ultimately produce) don't offer.

### Forecasting and Personalize
This talk was about the new AWS Forecasting and Personalize services.

[AWS AI](https://aws.amazon.com/machine-learning/) offerings fall into three categories:
* AI Services: services that solve a specific problem; Forecasting and Personalize fall under this
* ML Services: services for essentially creating your own AI service; SageMaker
* Infrastructure: providing the hardware for running ML

I was interested in Forecasting which fortunately was up first. Unfortunately it's still under preview so not openly available. I'm already using SageMaker and DeepAR so it sounds like this could automate a lot of that and additionally make other ML algorithms available. Potentially good as a starting off point but it sounds like we've already progressed beyond what it will offer.

Personalise was the second talk and is about making the Amazon recommendation engine available to everyone. When I was working in e-commerce this would have been much more interesting but isn't particularly relevant to the work Ecogy is doing in the energy sector.

## Forecasting Bootcamp
I also attended the Forecasting Bootcamp the day before the summit. This is a relatively new course from the sounds of it that didn't seem quite sure who it was trying to address, or maybe it was trying to cover everyone.

Having already explored SageMaker and the DeepAR algorithm I was hoping for a bit more of a deep dive into best practices and details on the configuration options. This sadly want the case.

The course was actually made up of four parts:
1. A broad background of ML, focussing on what makes Forecasting different
2. A general covering of SageMaker and DeepAR
3. A presentation on Gluon and MXNet, basically selling AWS's preferred ML framework
4. LSTNet

It was an intense amount of information to cram into a single day and I would have preferred more focus on DeepAR. The labs were pretty much just running a Jupyter notebook through, with one or two lines needing editing to complete. It was pretty hard to pick up useful information from this process.

For me it's been much more rewarding to do this on my own time using the online resources at home. Even so there were a number of points made that clicked and have given me some ideas to try.

## Summary
AWS do these events well. I wasn't overly taken with the Forecasting Bootcamp but did come away with useful realisations and ideas to try out. The summit itself was great, a useful way to catch up with the latest offerings from AWS and best practices when using then. There are so many services being announced and so much information available it can be really handy to have it condensed down into a one hour session, it's just hard to pick the sessions that you want to attend out of the considerable range on offer.
