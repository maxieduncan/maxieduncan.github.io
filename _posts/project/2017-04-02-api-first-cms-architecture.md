---
layout: post
category : projects
tagline: "API first CMS: Architecture"
tags : [api, cms, nodes, reactjs, swagger]
---
{% include JB/setup %}

# Architecture

### Local

I had a good idea of the final architecture I wanted to get to but started out building my two web applications as local instances with the temporary in memory API instance alongside them.

![Locale Architecture](/assets/posts/API CMS Local.png "Local Architecture")

This was enough for a PoC to show that the underlying concept could work and that it was worthwhile building it out further.

### Docker

The next step for me was creating Docker images so that the instances could easily be run in a clustered environment. This should have advantages down the line for CI/CD as well as scaling.

![Docker Architecture](/assets/posts/API CMS - Docker.png "Docker Architecture")

### AWS: Development

To create a proper and scalable solution I had to discard the API instance which was always just a temporary in memory solution for the PoC. This was done by the addition of AWS API Gateway, Lambda functions and DynamoDB to persist the content.

![AWS Development Architecture](/assets/posts/API CMS - AWS Development.png "AWS Development Architecture")

This architecture provides a simple system to develop against and sets us up nicely for a production configuration. I'm just running a single host in the ECS cluster that runs both the Document Editor instance and the web application (which is doubling as both the front end web application and the preview application). ECS allows us to configure scaling of the Docker images as we want.

The API is build entirely using AWS provided services which will automatically scale as required.

### AWS: Production

The production configuration would split the front end web application out from the editor application.

![AWS Production Architecture](/assets/posts/API CMS - AWS Production.png "AWS Production Architecture")

The editor would be have it's own read/write API endpoint, with caching disabled, to ensure that the content delivered is always up to date. The Docker image for the Document Editor ReactJS app (and potentially preview image) would be set up to run in multiple availability zones for reliability.

The front end web application would be the one expected to see significant load as it delivers the content for end users of the live site. As such it would use ELB for load balancing and auto scaling to ensure that enough web application instances are running to handle the current load across multiple availability zones for redundancy. It would also have it's own API end point, providing a read only API with caching enabled to reduce the cost of the API gateway.
