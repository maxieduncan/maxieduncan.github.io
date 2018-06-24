---
layout: post
category : aws
title: "Sharing API Gateway and Authorizer"
tags : [aws, apigateway, lambda, serverless, cloudformation]
---
{% include JB/setup %}

I’ve been using [serverless](https://serverless.com/) to deploy an API Gateway and recently hit the AWS limit of  200 resources in a CloudFormation template. This wasn’t entirely unexpected and my code base is already split up into logical components that I can easily break my API up into.

The main requirement I have is that I want to keep all my endpoints under a single API Gateway. Serverless does support this and breaking up my `serverless.yml` into the logical components that [share an API Gateway](https://serverless.com/framework/docs/providers/aws/events/apigateway/#share-api-gateway-and-api-resources) was relatively straight forward.

I did encounter issues with the Cognito User Pool Authorizer and sharing it across the API Gateway. Fortunately, support to [share an API Authorizer](https://serverless.com/framework/docs/providers/aws/events/apigateway/#share-authorizer) has recently been added to serverless though I found the example lacking.

### Example
The following code example is an example of how to share an API Gateway and Authorizer in multiple `serverless.yml` definitions supporting servers stages.

#### Creating the Shared API Gateway and Authorizer

```
service: api-gateway

resources:
  Resources:
    SharedApiGateway:
      Type: AWS::ApiGateway::RestApi
      Properties:
        Name: api-${opt:stage, self:provider.stage}

    SharedApiGatewayAuthorizer:
      Type: AWS::ApiGateway::Authorizer
      Properties:
        AuthorizerResultTtlInSeconds: 300
        IdentitySource: method.request.header.Authorization
        Name: api-auth-cognito-${opt:stage, self:provider.stage}
        RestApiId:
          Ref: SharedApiGateway
        Type: COGNITO_USER_POOLS
        ProviderARNs:
          - ${env:USERPOOL_ARN}

  Outputs:
    apiGatewayRestApiId:
      Value:
        Ref: SharedApiGateway
      Export:
        Name: apiGateway-restApiId-${opt:stage, self:provider.stage}

    apiGatewayRestApiRootResourceId:
      Value:
         Fn::GetAtt:
          - SharedApiGateway
          - RootResourceId
      Export:
        Name: apiGateway-rootResourceId-${opt:stage, self:provider.stage}

    apiGatewayAuthorizerId:
      Value:
        Ref: SharedApiGatewayAuthorizer
      Export:
        Name: apiGateway-authorizerId-${opt:stage, self:provider.stage}

provider:
  name: aws
  apiGateway:
      restApiId:
        Ref: SharedApiGateway
      restApiResources:
        Fn::GetAtt:
            - SharedApiGateway
            - RootResourceId

```

The API Gateway and Authorizer are created in this definition and their IDs are exported (using the stage name for uniqueness) so that they can be used in the sub services.

#### Using the Shared API Gateway and Authorizer
```
service: api-gateway-service-a

provider:

  apiGateway:
    restApiId:
      'Fn::ImportValue': apiGateway-restApiId-${opt:stage, self:provider.stage}
    restApiRootResourceId:
      'Fn::ImportValue': apiGateway-rootResourceId-${opt:stage, self:provider.stage}
functions:


  # Project endpoints
  functionA:
    handler: functionA
    events:
      - http:
          path: a
          method: get
          cors: true
          authorizer:
            type: COGNITO_USER_POOLS
            authorizerId: ${cf:ecogy-portfolio-service-${opt:stage, self:provider.stage}.apiGatewayAuthorizerId}
```

The main points here are that the `apiGateway-restApiId` and the `apiGateway-rootResourceId` exported in the shared definition are imported here so that we can add our functions to the shared API Gateway.

When referencing the Authorizer, importing the ID value didn’t work as it did for the API Gateway IDs; instead the Cloud Formation template and the output property `apiGatewayAuthorizerId` has to explicitly referenced.
