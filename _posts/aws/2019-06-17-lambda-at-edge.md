---
layout: post
category : aws
title: "Lambda@Edge"
tags : [aws, lambda, serverless]
---
{% include JB/setup %}

I was recently asked to set up an S3 bucket with basic authentication enabled over https. This is not a configuration offered by AWS but there is a nice [blog post](https://hackernoon.com/serverless-password-protecting-a-static-website-in-an-aws-s3-bucket-bfaaa01b8666) that shows you how to do this using [Lambda@Edge](https://aws.amazon.com/lambda/edge/).

We're using [serverless](https://serverless.com) to deploy our AWS resources and while the configuration is pretty well documented there were some unexpected behaviours from running Lambda@Edge that I wasn't aware of, namely:
* The CloudWatch logs are recorded in the region that the CloudFront deployment is accessed on; additionally these logs have a different path from the deployed function (presumably so as not to conflict if the lambda function is deployed directly into that region).
* Environment variables are not available to Lambda@Edge functions

Both of these are actually reasonable and sensible restrictions and clearly documented but it wasn't something I was aware of going into this and the logging in particular made things harder to track down.

In addition I had the lambda permissions restricted to the region I deployed the function into, not taking into account that Lambda@Edge meant it would be accessed from a different region. In the end I chose to make the function explicitly set this region no matter where it was being called from.

### Debugging Lambda@Edge
[Testing and Debugging Lambda@Edge Functions - Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-edge-testing-debugging.html#lambda-edge-testing-debugging-determine-region)

There's code provided by AWS that allows you determine the region your Lambda@Edge function is executing in as well as the path of the log:
```
FUNCTION_NAME=function_name_without_qualifiers
for region in $(aws --output text  ec2 describe-regions | cut -f 3)
do
    for loggroup in $(aws --output text  logs describe-log-groups --log-group-name "/aws/lambda/us-east-1.$FUNCTION_NAME" --region $region --query 'logGroups[].logGroupName')
    do
        echo $region $loggroup
    done
done
```

The output for this looks like:
```
us-east-1 /aws/lambda/us-east-1.ecogy-repository-debian-int-basicAuth
us-west-2 /aws/lambda/us-east-1.ecogy-repository-debian-int-basicAuth
```
letting you know the `region` and the `path` of the log in CloudWatch.

### Lambda@Edge restrictions
[Requirements and Restrictions on Lambda Functions - Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-requirements-limits.html#lambda-requirements-lambda-function-configuration)

I wanted the basic auth credentials to be configurable and the first option I tried using was environment variables, something it quickly became apparent didn't work but again clearly documented.

My solution to this was to retrieve the details direct from the Parameter Store, something which worked great as soon as I got the region permissions worked out.

### The code

The serverless configuration is straight forwards, making use of the [serverless-plugin-cloudfront-lambda-edge](https://github.com/silvermine/serverless-plugin-cloudfront-lambda-edge) plugin to handle the versioning that is required by Lambda@Edge of the lambda functions.

```
/# Ecogy protected repository for debian packages hosted on S3/

service: ecogy-repository-debian

plugins:
  - "@silvermine/serverless-plugin-cloudfront-lambda-edge"
  - serverless-pseudo-parameters

provider:
  name: aws
  region: us-east-1
  runtime: nodejs8.10

  iamRoleStatements:
    - Effect: Allow
      Action:
        - ssm:GetParameter
      Resource: arn:aws:ssm:#{AWS::Region}:#{AWS::AccountId}:parameter/REPOSITORY/DEBIAN/*

functions:
  basicAuth:
    handler: src/handler.basicAuth
    memorySize: 128 /# limit for lambda @ edge/
    timeout: 5 /# limit for lambda @ edge/
    lambdaAtEdge:
      distribution: DebianRepositoryDistribution
      eventType: viewer-request

resources:
  Resources:
    S3BucketDebianRepository:
      Type: AWS::S3::Bucket
      Properties:
        AccessControl: Private
        BucketName: ecogy-debian-repo-${opt:stage}
        PublicAccessBlockConfiguration:
          BlockPublicAcls: true
          BlockPublicPolicy: true
          IgnorePublicAcls: true
          RestrictPublicBuckets: true

    DebianRepositoryBucketPolicy:
      Type: "AWS::S3::BucketPolicy"
      Properties:
        Bucket:
          Ref: S3BucketDebianRepository
        PolicyDocument:
          Version: "2012-10-17"
          Statement:
            - Action:
                - "s3:GetObject"
              Effect: "Allow"
              Resource:
                Fn::Join:
                  - ""
                  - - "arn:aws:s3:::"
                    - Ref: S3BucketDebianRepository
                    - "/*"
              Principal:
                AWS:
                  {
                    "Fn::Join":
                      [
                        "",
                        [
                          "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity ",
                          {
                            Ref: DebianRepositoryCloudFrontOriginAccessIdentity,
                          },
                        ],
                      ],
                  }

    DebianRepositoryDNSRecord:
      Type: AWS::Route53::RecordSet
      Properties:
        HostedZoneId: ${ssm:/CodeBuild/HOSTED_ZONE-${opt:stage}}
        Name: debian.repo.${ssm:/CodeBuild/BASE_DOMAIN-${opt:stage}}
        Type: A
        AliasTarget:
          HostedZoneId: Z2FDTNDATAQYW2 /# Cloudfront/
          DNSName:
            "Fn::GetAtt":
              - DebianRepositoryDistribution
              - DomainName

    DebianRepositoryCloudFrontOriginAccessIdentity:
      Type: AWS::CloudFront::CloudFrontOriginAccessIdentity
      Properties:
        CloudFrontOriginAccessIdentityConfig:
          Comment: DebianRepositoryCloudFrontOriginAccessIdentity

    DebianRepositoryCertificate:
      Type: AWS::CertificateManager::Certificate
      Properties:
        DomainName: debian.repo.${ssm:/CodeBuild/BASE_DOMAIN-${opt:stage}}

    DebianRepositoryDistribution:
      Type: AWS::CloudFront::Distribution
      Properties:
        DistributionConfig:
          Aliases:
            - debian.repo.${ssm:/CodeBuild/BASE_DOMAIN-${opt:stage}}
          Origins:
            - Id: S3-ecogy-debian-repo-${opt:stage}
              DomainName: ecogy-debian-repo-${opt:stage}.s3.amazonaws.com
              S3OriginConfig:
                OriginAccessIdentity:
                  {
                    "Fn::Join":
                      [
                        "",
                        [
                          "origin-access-identity/cloudfront/",
                          {
                            Ref: DebianRepositoryCloudFrontOriginAccessIdentity,
                          },
                        ],
                      ],
                  }
          DefaultCacheBehavior:
            TargetOriginId: S3-ecogy-debian-repo-${opt:stage}
            ViewerProtocolPolicy: redirect-to-https
            MinTTL: 0
            AllowedMethods:
              - HEAD
              - GET
              - OPTIONS
            CachedMethods:
              - HEAD
              - GET
            SmoothStreaming: false
            DefaultTTL: 3600
            MaxTTL: 31536000
            Compress: false
            ForwardedValues:
              QueryString: false
          PriceClass: PriceClass_100
          Enabled: true
          ViewerCertificate:
            AcmCertificateArn:
              Ref: DebianRepositoryCertificate
            SslSupportMethod: sni-only
            MinimumProtocolVersion: TLSv1.1_2016


```

The function itself is mostly lifted from the Hacker Noon blog post, the primary changes being the use of the Parameter Store and explicitly setting the region for when the function is called at the edge from another region:

```
"use strict"

const AWS = require("aws-sdk") // eslint-disable-line import/no-extraneous-dependencies
AWS.config.update({ region: "us-east-1" })
const ssm = new AWS.SSM()

exports.basicAuth = async (event, context, callback) => {
  try {
    // Configure authentication
    const authUser = await ssm
      .getParameter({ Name: "/REPOSITORY/DEBIAN/USER", WithDecryption: true })
      .promise()
      .then(data => {
        return data.Parameter.Value
      })
    const authPass = await ssm
      .getParameter({ Name: "/REPOSITORY/DEBIAN/SECRET", WithDecryption: true })
      .promise()
      .then(data => {
        return data.Parameter.Value
      })

    // Get request and request headers
    const request = event.Records[0].cf.request
    const headers = request.headers

    // Construct the Basic Auth string
    const authString =
      "Basic " + new Buffer(authUser + ":" + authPass).toString("base64")

    // Require Basic authentication
    if (
      typeof headers.authorization == "undefined" ||
      headers.authorization[0].value != authString
    ) {
      const body = "Unauthorized"
      const response = {
        status: "401",
        statusDescription: "Unauthorized",
        body: body,
        headers: {
          "www-authenticate": [{ key: "WWW-Authenticate", value: "Basic" }]
        }
      }
      callback(null, response)
    }

    // Continue request processing if authentication passed
    callback(null, request)
  } catch (err) {
    console.log("Error while verifying basic auth header")
    console.log(err)
    throw err
  }
}

```

Setting the region explicitly like this admittedly makes Lambda@Edge a little pointless as it's always executing in the one region; a better solution would be to configure cross region permissions. The reason for using Lambda@Edge wasn't for the improved performance in this case though, rather it was to implement the basic authentication which this configuration did.

In the end it turned out we didn't actually need to support basic authentication at all on the S3 bucket, there was a plugin that allowed the bucket to be accessed using S3 credentials directly and the CloudFront distribution and Lambda@Edge were no longer required.
