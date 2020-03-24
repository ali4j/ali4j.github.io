---
layout: post
title: "Prerequisites of Kubernetes"
date: 2020-3-24 19:00:00 +0400
description: some required concepts about containerization.
img: prerequisites_of_kubernetes.jpg
tags: [docker,kubernetes, k8s, container, image]

---
Kubernetes aka k8s is an open source system developed by google and it is the most famous containers orchestration tool in the industry at the moment. Before getting into it, it’s a better idea to take a look at containers.

## Containers


### The need for containers
There are software systems which are consist of several different components, for example consider an application which uses redis as messaging infrastructure, mongodb as database, nignx as web server and nodejs as backend technology.

![technology_stack_image]({{site.baseurl}}/assets/img/technology_stack_image.jpg)

The compatibility of these products should be checked with the OS which based on it the software system is going to be deployed. Also the problem gets more confusing when different operating systems are used in different environments (for instance, windows is used in development environment and linux is used in production).

In addition to the above problem, the compatibility of these products should be checked against the libraries and dependencies on the OS. Different services require different libraries which different versions. This issue makes the problem more complicated.
Also during time new versions of services and components are released by their vendors and it is required to upgrade existing ones. During the upgrading process we have to go trough above mentioned problems and it makes it a quite tedious and time-consuming job. These compatibility issue problems are called the matrix from hell.

Finally whenever a new developer joins the team, considering the components and services used in the project, setting up the development environment sounds quite challenging and definitely he/she will have a hard time setting up the dev environment. This is another reason which makes the usage of containers a must.


### The Magic

There should be some thing which addresses the problem of compatibility, some kind of tool which makes it possible to change components and services without affecting other ones. Docker is the platform which is able to handle these issues easily. Docker makes its possible to run each component and all of its libraries and dependencies inside a separate environment called container. There should only be a docker configuration file and every developer by building this file is capable of setting up the dev environment fast and easy and the only requirement in this scenario is that docker engine should be installed.


### Containers, The real Magic

Containers are isolated environments which can have their own processes, mounts, network interfaces and … The only (and the most important) difference between containers and virtual machines is that they share the kernel of the host operating system, while in virtualization each virtual machine has its own fully-fledged OS.

Operating systems are consists of two parts. First is the kernel and the second one is the utility softwares. For example, there are several linux distros and the fact about them is that they share the same kernel (the linux kernel) and they only differ in utility softwares. The kernel is responsible for talking to the underlying hardware and utility softwares are based on the kernel.

![docker_os_image]({{site.baseurl}}/assets/img/OSs.jpg)

As it is mentioned, containers share the kernel of the host OS.  It means that containers only have utility softwares and they use the kernel of the host OS. From these reason the type of the containers and the host OS should be the same. For example, when docker engine is installed on top of ubuntu, it is not possible to have a windows container and they should be based on linux containers such as centos, fedora and etc.

![tech_stack_on_docker_image]({{site.baseurl}}/assets/img/tech_stack_on_docker_image.png)

This is not considered as a pitfall since the main idea behind docker is running containers as separated environments while sharing the kernel of the host operating system, and it is due to this fact that containerization sometimes is referred to as light virtualization.


### Containers VS Virtual Machines

The main differences between these two technologies are listed below:

* Containers use host OS kernel, while vms have their own operating machines, due to this overhead vms consume resources remarkably more than containers. 

* Also vms are larger than containers in size, they take gigabytes of disk while containers size are in megabytes.

* In addition, the boot up speed in containers is faster than vms, since vms should start up an entire OS.

* Finally, because vms are isolated from the host OS, they are capable of running every kind of operating system regardless of the host OS, while containers should be the same as host OS.



### How the magic happens?

Almost all of the software vendors have containerized they products and made them publicly available. They share the image of their products using docker registry (or docker hub) and with just executing a single command a developer can run an instance of that product on their machine. For example by executing `docker run tomcat` an instance of apache tomcat will be running on docker. Also by executing the command again another instance will be running on docker and it makes clustering quite easy. There are tools for handling such cases and they are going to be discussed in next paragraphs.


### Containers VS Images

Image is a template which is used to create one or more containers. Containers are running instances of images and several containers can be built by one image. As it is said there are official images of software products publicly available but in cases when desired images are not available it is possible to create custom images. For example, a developer can create their own image of the application they are working on and by sharing them which devops guys makes it possible for them to run the application easily.


### Conclusion

In this article we scratched the basics of containerization. This article can be considered as an overview of docker which is required for containerization orchestration.
In next article we will take a look at  containerization orchestration basics.