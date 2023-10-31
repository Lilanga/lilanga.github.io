---
title: Containerizing applications, create container images with Dockerfile and Cloud Native Buildpacks
date: 2023-10-30 16:00:00 +0800
categories: [Optimisations, Performance, Containerization, Docker, Cloud Native Buildpacks, ]
tags: [docker, containerization, cloud native buildpacks]
render_with_liquid: true
mermaid: true
image:
  path: /assets/img/posts/2023-10-30/cover.png
---

Containerization is the technique that helps developers package and run applications in isolated environments called containers. Containers are lightweight, portable, and consistent, which makes them ideal for deploying applications across different platforms and environments.
s
Docker is the widely accepted platform for creating and managing containers. Container images are the building blocks for docker containers. A container image is a file that contains everything needed to run an application in a container, such as the code, libraries, dependencies, and configuration.

In this blog post, Let's compare and contrast different methods for creating container images. We will focus on two generic container creation approaches, namely [Dockerfile](https://docs.docker.com/engine/reference/builder){:target="_blank"}, [Cloud Native Buildpacks](https://buildpacks.io){:target="_blank"}. There are numerous approaches to containerize applications using language-specific tools such as [Jib](https://github.com/GoogleContainerTools/jib){:target="_blank"} for Java and [Ko](https://ko.build){:target="_blank"} for Golang as well. We will also discuss their strengths and weaknesses and recommend when to use them in a different write-up.

## How Docker and Build Pack were born, a bit of history to Containerization

The history of Containerization using Docker and build packs can be traced back to the early 2010s, when two competing cloud hosting companies, [Heroku](https://www.heroku.com){:target="_blank"} and [dotCloud](https://www.docker.com/press-release/dotcloud-inc-now-docker-inc/){:target="_blank"}, were developing their in-house solutions for fast and optimal application delivery mechanisms.

Heroku used Linux Containers ([LXC](https://linuxcontainers.org){:target="_blank"}), a technology that uses kernel features to isolate processes and resources on a Linux system. Heroku created a build system with LXC and evolved it to form build packs, which are scripts that define how to install and configure an application's dependencies for different types of application stacks.

DotCloud, on the other hand, was working on a more innovative virtualization system using LXC as a base but added features such as image management, networking, portability, and orchestration. This system was created by [Solomon Hykes](https://twitter.com/solomonstre){:target="_blank"}, the founder and CTO of dotCloud. He initially built it as an in-house tool to make it easier to deploy applications on their dotCloud platform.

### The Origin of Docker

In 2013, Hykes decided to open-source the project and rename it Docker. The concept of shipping software as containers inspired many around the world. Docker quickly gained popularity among developers and cloud providers, offering a simple and universal way to package and run applications in any environment. Docker also became an industry standard for containers, with many other tools and platforms adopting or integrating it.

### The Origin of Cloud Native build packs

[Pivotal](https://tanzu.vmware.com/pivotal){:target="_blank"}, which was spun off from VMware in 2013, adopted Heroku build packs for its Cloud Foundry platform but made some modifications and improvements to suit its own needs. Pivotal's build packs used Garden, a custom container engine, instead of LXC, and added features such as caching, staging, and health checks.

In January 2018, Heroku and Pivotal decided to join forces and create a new project called Cloud Native Buildpacks, with the goal of unifying the buildpack ecosystems and creating a standard for building container images. Cloud Native Buildpacks joined the Cloud Native Computing Foundation (CNCF) in October 2018 as a sandbox project and moved to the incubating stage in November 2020

Cloud Native Buildpacks use the OCI image format as the output format, which makes them compatible with any container runtime or registry that supports OCI. Cloud Native Buildpacks also use a modular design that allows different components to be swapped or customized, such as the lifecycle, the platform, and the builder. Cloud Native Buildpacks also leverage some of the latest features of Docker API v2, such as cross-repository blob mounting and image layer rebasing, to optimize the image creation and update process.

## What is a Container Image

![Container image internals](/assets/img/posts/2023-10-30/container-image.png)

A container image is a file that contains everything needed to run an application in a container. It is a tar file bundling everything needed to create a runtime environment for your application. Sometimes, we state the container image as a snapshot of the running container. It is a blueprint to create a runnable sandbox environment for your application, a running container.

Container images are stored and distributed using registries and repositories. A registry is a service that hosts and serves container images over the network. A repository is a collection of related images within a registry, usually identified by a name and a tag. For example, docker.io/library/ubuntu:latest is an image in the Ubuntu repository with the latest tag in the docker.io registry.

A container image consists of two main parts: layers and metadata.

### Layers

Layers are the building blocks of a container image. Each layer represents a change or an addition to the image, such as installing a package, copying a file, or running a command.

> Layers are stacked on top of each other in a specific order to form the final image.
{: .prompt-tip }

### Metadata

Metadata is the information that describes the image and its layers, such as the name, version, author, and labels. Metadata also includes instructions on how to run the image as a container, such as the entry point, command, environment variables, ports, and volumes.

Container images are stored and distributed using registries and repositories. A registry is a service that hosts and serves container images over the network. A repository is a collection of related images within a registry, usually identified by a name and a tag.

> For example, docker.io/library/ubuntu:latest is an image in the Ubuntu repository with the latest tag in the docker.io registry.
{: .prompt-info }

## Approaches to create container images

![Approaches to build container](/assets/img/posts/2023-10-30/build-container.png)

There are many ways to create container images. Dockerfile and Cloud Native Buildpacks are the most popular generic approaches to create them. There are language specific approaches such as [Jib](https://github.com/GoogleContainerTools/jib){:target="_blank"} for Java and [Ko](https://ko.build){:target="_blank"} for Golang. Docker containerization has also influenced the development of other container technologies, such as [cri-o](https://cri-o.io){:target="_blank"}, [containerd](https://containerd.io){:target="_blank"} and [Kubernetes](https://kubernetes.io){:target="_blank"} and create tooling like [podman](https://podman.io){:target="_blank"}, [buildah](https://buildah.io){:target="_blank"}, [kaniko](https://github.com/GoogleContainerTools/kaniko){:target="_blank"}, etc. Most of those ecosystem tools are using Dockerfile as a base to create container images.

### Dockerfile

A Dockerfile is a text file that contains a series of instructions to build a container image. Each instruction corresponds to a layer in the image. The instructions are executed in order from top to bottom by the docker build command.

Here is an example of a simple Dockerfile for a Java application:
  
```dockerfile
# Use an official Java runtime as a base image
FROM openjdk:11-jre-slim

# Set the working directory
WORKDIR /app

# Copy the application jar file to the working directory
COPY target/app.jar app.jar

# Expose port 8080
EXPOSE 8080

# Run the application
CMD ["java", "-jar", "app.jar"]
```

The Dockerfile starts with a FROM instruction that specifies the base image to use. The base image is the starting point for the image build process. It can be any image in a registry or repository, such as docker.io/library/openjdk:11-jre-slim.

Dockerfiles are easy to write and understand as they give you sequential execution order of the instructions. It is also easy to debug and troubleshoot. Dockerfiles give complete control and flexibility to the developer to create the container image. It is also well integrated with other tools and platforms, such as Kubernetes or CI/CD pipelines. Dockerfiles uses caching and reusing layers to speed up the build process.

Dockerfiles are also easy to misuse. It is easy to create a large image with many layers and dependencies. It is also easy to create a container image with security vulnerabilities. Dockerfiles are also not portable across different platforms and environments. It is also not easy to create a container image for a multi-stage build. Dockerfiles are also not easy for non-developers and require manual maintenance, patching, and updates.

### Cloud Native Buildpacks

Buildpacks are a framework that automates the creation of container images for applications. Buildpacks detect the language and framework of the application, install the required dependencies, and configure the runtime environment.

> Buildpacks produce images that adhere to best practices and standards for security, performance, and compatibility.
{: .prompt-tip }

Here is an example of using Buildpacks to create a container image for the same Java application:

```bash
# Install the Pack CLI tool
$ curl -L https://github.com/buildpacks/pack/releases/download/v0.21.1/pack-v0.21.1-linux.tgz | tar xz -C /usr/local/bin

# Create an image using the default builder
$ pack build app --path target/app.jar

```

Developer experience is the main advantage of using Buildpacks. Buildpacks are easy to use and require minimal configuration. Buildpacks are also easy to maintain and update. Buildpacks are also portable across different platforms and environments. Buildpacks are also easy to use for non-developers and require minimal manual maintenance, patching, and updates.

Buildpacks optimize the container image creation process by using caching and reusing layers. This process also optimizes the container image size by removing unnecessary dependencies and files. Buildpacks also optimize the container image security by scanning for vulnerabilities and applying security patches.

On the other hand, build packs are challenging to debug and troubleshoot. Buildpacks are also not easy to customize and extend, with their limited flexibility options. Buildpacks has limited support with other tools and platforms out of the box. Buildpacks may only support some languages and frameworks out of the box as well. With build packs, multi-stage builds are challenging to achieve, and build systems are not designed to utilize multi-stage build systems.

## Comparison between Dockerfile and Buildpacks

Following table compares and contrasts Dockerfile and Buildpacks based on number of different criteria.

| Criteria | Dockerfile | Buildpacks |
| :--- | --- | --- |
| **Developer Experience** | Easy to write and understand, <br> may challenging to use, extend and maintenance | Easy to use |
| **Maintenance** | Manual maintenance, patching, and updates | Automatic maintenance, patching, and updates |
| **Security** | No security checks, <br> Requires careful handling <br> of sensitive data and permissions, <br> as well as regular updates and patches | Ensures security and compliance by applying <br> patches and updates regularly,<br> as well as using trusted builder images |
| **Performance** | No optimizations out of the box | Optimizes the image size and layers automatically <br> by separating the application layer from the dependency layer |
| **Portability** | Not portable | Portable |
| **Extensibility** | Easy to customize and extend | Limited customization and extension |
| **Debugging** | Easy to debug and troubleshoot | Hard to debug and troubleshoot |
| **Integration** | Well integrated with other tools and platforms | Limited integration with other tools and platforms |
| **Multi-stage builds** | Easy to create multi-stage builds | Hard to create multi-stage builds |
| **Non-developers** | Hard to use for non-developers | Easy to use for non-developers |

## Summary

In this blog post, we have discussed two generic approaches to creating container images for applications, namely Dockerfile and Cloud Native Buildpacks. A Dockerfile is a text file that contains a series of instructions to build a container image. Many tools and frameworks were created later that support Dockerfile as container specification apart from the docker tool. Buildpacks are a framework that automates the creation of container images for applications. Buildpacks detect the language and framework of the application, install the required dependencies, and configure the runtime environment. At the same time, Dockerfile gives you complete control and flexibility to create the container image according to your needs.
