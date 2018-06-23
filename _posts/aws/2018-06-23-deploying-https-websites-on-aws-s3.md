---
layout: post
category : aws
title: "Deploying HTTPS websites on AWS S3"
tags : [aws, s3, cloudfront, route53, react]
---
{% include JB/setup %}

[Hosting a static website on AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html) is pretty straightforward, adding HTTPS  with a custom domain to the mix is a little more complex and requires using CloudFront and a number of configuration options and services.

The main steps to hosting a static website on S3 over HTTPS are:
* [register a custom domain using Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/registrar.html)
* [create a bucket with that custom domain](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html)
* [use CloudFront to cache your website](https://docs.aws.amazon.com/AmazonS3/latest/dev/website-hosting-cloudfront-walkthrough.html)

It’s setting up CloudFront where I found knowing what needed to be configured a little more complex and hit a few gotchas.

### SSL Certificate
You’ll need to have an SSL certificate set up for your custom domain to enable HTTPS access. Use [AWS Certificate Manager](https://aws.amazon.com/certificate-manager) to create an SSL certificate for the domain you set up in Route53.

### Configuration
`Alternate Domain Names (CNAMEs)` : the custom domain name you set up in Route53 and that matches your bucket name
`SSL Certificate`: the SSL certificate that you set up for your custom domain
`Default Root Object`: the index page of your web app, typically `/index.html`
`Viewer Protocol Policy`: use to restrict access to HTTPS as desired

### Caching
By default CloudFront will cache content for 24 hours. If you want to be able to deploy changes to your website on short notice then you’ll want to reduce this.

### Error Pages
Using [React](https://reactjs.org/) and [React Router](https://reacttraining.com/react-router/web/guides/philosophy) I encountered issues when directly accessing URLs that weren’t the root, as the main application hadn’t been loaded so a 404 was generated.

The “fix” for this, and it is a bit of a hack, is to configure the error pages on your CloudFront distribution to use your `/index.html`. I configured the `403` and `404` pages to respond with `/index.html` and a status code of `200`. This means that the index page gets loaded for these requests and React Router can then kick in and handle the requests.

### Route53
In Route53 I set up an A record for the custom domain with an Alias to the CloudFront distribution.
