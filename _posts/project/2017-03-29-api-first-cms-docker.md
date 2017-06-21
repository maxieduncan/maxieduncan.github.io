---
layout: post
category : projects
tagline: "API first CMS: Docker"
tags : [api, cms, nodes, reactjs, swagger]
---
{% include JB/setup %}

# Docker

Making the two React applications into Docker images makes them easy to deploy and scale as required. This is quite simple to do at the moment as they're stateless applications but in reality this wouldn't be the case and is something that I hope to investigate more if I implement user accounts.

The Document Editor probably won't need to scale as the you wouldn't expect the load on the management application to be high; however being able to add it to a cluster with Multi AZ support will make it more fault tolerant. The web application on the other hand most likely could benefit from being able to scale automatically to handle peaks in demand and, it would also gain from the fault tolerance gained by Multi AZ support. Of course this can be done by running the applications natively on EC2 instances but the Docker images are easier to manage.

### Dockerfile

Creating the Docker images was straightforward, just a matter of adding a Dockerfile to the root of the application to copy the appropriate contents and configure the software to be installed and the start up command to run. The addition of a .dockerignore file allows you to exclude files that shouldn't be copied onto the image.

Once the Dockerfile is properly configured its just a matter of building the image:

`docker build -t cms-poc/document-editor .`

then running it:

`docker run -p 3000:3000 -d cms-poc/document-editor`

At this point rather than running the stacks natively I've got them up and running in Docker and everything works as expected.

### EC2 Container Service

The next step was to get these images up and running on AWS which provides the EC2 Container Service for just this purpose. This takes care of configuring and running the host EC2 instances for you and offers configuration options for load balancing and autoscaling. At this point I'm just interested in getting the images running and keeping the cost down so I stick to a single t2.micro instance.

It's also at this point that I hit the first (expected) issue, my images have localhost embedded in a couple of files and can't resolve the endpoints correctly. This is solved easily enough by changing the configuration in the NodeJS application to retrieve these values from the environment variables which can be passed to the Docker instance. The complication for me here was that I was running React and it filters out custom variables that aren't prefixed with REACT_APP_, so I spent a frustrating couple of hours trying to work out why they weren't appearing before reading the documentation I should have in the first place. Once the correct prefix was in place I was able to pass the endpoint URLs into the Docker instances using the environment variables and have everything connect as expected.

So the web applications are now up and running and can easily be updated and configured to use load balancing and autoscaling.

### Final Thoughts

Using Docker and getting my applications up and running was pretty easy but the configuration I have isn't one you'd use in production.

The React applications are both running the development builds, which makes it possible to dynamically pass the in the URLs of the services they connect to via environment variables. In reality you'd use a production build that creates a static website, so if using Docker you'd need to create environment specific images that had the appropriate URLs embedded in the image; or you might just hard code a single URL and use DNS to resolve to the appropriate service. The choice would depend on the actual requirements. For a more typical dynamic app, the environment variables would be the correct solution.

Introducing state would be interesting as well, if you're in a cluster you can't necessarily depend on the same instance answering every request. There are ways around this of course, using an in memory data store such as Memcached or Redis, persisting to a DB or even storing data in Cookies. It remains to be seen if that's something this application will require, at the moment it certainly doesn't.
