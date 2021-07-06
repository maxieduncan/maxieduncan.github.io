---
layout: post
category: aws
title: "Lambda Container Image Support"
tags: [aws, lambda, serverless, docker, nodejs]
---

{% include JB/setup %}

I recently had a requirement to generate PDF reports on a scheduled basis and developed a lambda function in NodeJS with [pdfkit](https://pdfkit.org/) to do just that. Some graphing was required so I made us of [Chart.js](http://www.chartjs.org/) with [chartjs-node-canvas](https://www.npmjs.com/package/chartjs-node-canvas) allowing me to do this in a headless environment without jsdom.

This all worked perfectly until I tried deploying the lambda function I’d been developing locally on my Mac with [serverless](https://www.serverless.com/) to AWS. At this point things fell over with the error: `Error: libuuid.so.1: cannot open shared object file: No such file or directory` . Some googling took me to an open [Github issue](https://github.com/Automattic/node-canvas/issues/1448) explaining that the issue is caused by a library not being present in the AWS Lambda image.

The problem is that `canvas` requires natively compiled libraries, these were available when I ran `npm install` on my Mac but weren’t available when serverless deployed the same package to AWS lambda. Thankfully it seemed the same open issue had an answer, the person who had reported the issue had also created a lambda layer [ jwerre/node-canvas-lambda](https://github.com/jwerre/node-canvas-lambda) that made the missing libraries available, a very cool solution! Except it didn’t work for me. The compiled version provided was too old and when I tried to create the layer myself all the dependencies were missing.

This had however put me on the right path, now that I knew what the issue was I could make use of a feature that AWS had introduced recently: [AWS Lambda – Container Image Support](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/). This allowed me to use a custom container image and install the additional dependencies that I required. This proved to be very easy, simply adding a Dockerfile that created a container from the AWS lambda NodeJS image and added the additional dependencies. Serverless made it easy to just point my function at the custom container rather than the local function and like that it just worked, the function I’d been deploying was now just bundled up into the custom container which had the additional dependencies and they were deployed together. Problem solved. The only annoyance is that image based functions like this can’t be run locally by serverless.

Dockerfile:

```
FROM public.ecr.aws/lambda/nodejs:14

# set up container
RUN yum -y update \
&& yum -y groupinstall "Development Tools" \
&& yum install -y nodejs gcc-c++ cairo-devel libjpeg-turbo-devel pango-devel giflib-devel

RUN npm install --build-from-source canvas@next
RUN npm install \
chartjs-node-canvas \
chart.js

COPY src/ package* ./
RUN npm install

```

serverless.yml snippet:

```
provider:
  name: aws
  stage: dev
  runtime: nodejs14.x
  ecr:
    # In this section you can define images that will be built locally and uploaded to ECR
    images:
      appimage:
        path: ./

functions:
  generateReport:
    image:
      name: appimage
      command: src/handler.generateReport

```
