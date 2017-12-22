---
layout: post
title: "Getting Started with Kubernetes using Minikube"
description: "An introduction to Kubernetes using Minikube. Learn how to run single node Kubernetes cluster locally"
date: 2017-12-20 02:54
category: Technical Guide, DevOps
author: 
  name: "O'Brian Kimokot"
  url: "https://twitter.com/O_BRIANKIM"
  mail: "bkim19302@gmail.com"
  avatar: "https://gravatar.com/avatar/00af6e085efbc34b55b184c779601fe2?s=200"
design: 
  bg_color: "#00a7cc"
  image: "https://cdn.auth0.com/blog/python-restful/logo.png"
tags: 
- docker
- minikube
- kubernetes
- docker
- virtualbox
---

**TL;DR:** In this tutorial, we will learn how to deploy a simple application locally using Kubernetes and Minikube. These two tools have a lot of concepts to
learn about but we will focus on the main and most important to carry out a minimal deployment.

# Introduction to Kubernetes

Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications. It groups containers that make up an application into logical units for easy management and discovery. Kubernetes brings the following ideas and practices to the world of DevOps.

- Planet Scale - Kubernetes is designed on the same principles that allows Google to run billions of containers a week, Kubernetes can scale without increasing your ops team.

- Never outgrow - Kubernetes flexibility grows with to deliver your applications consistently and easily no matter how complex the need is.

- Run anywhere - Kubernetes is open source giving the freedom to take advantage of on-premises, hybrid, or public cloud infrastructure, letting you to effortlessly move workloads to where it matters to you.

# Kubernetes Features

- Horizontal scaling - Scale your application up and down with a simple command, with a UI, or automatically based on CPU usage.
- Automated rollouts and rollbacks - Kubernetes progressively rolls out changes to your application or its configuration, while monitoring application health to ensure it doesn't kill all your instances at the same time. If something goes wrong, Kubernetes will rollback the change for you.
- Self-healing - Restarts containers that fail, replaces and reschedules containers when nodes die, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.
- Service discovery and load balancing - Kubernetes gives containers their own IP addresses and a single DNS name for a set of containers, and can load-balance across them.
- Secret and configuration management - Objects pf type `secret` are used to hold sensitive information like passwords, OAuth tokens, and ssh keys. Kubernetes allows us to deploy and update secrets and application configuration without rebuilding our images.

These ate just some of the main features that we get when using Kubernetes.

# What is Minikube

Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a VM on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

Alternatives to using Minikube include deploying applications to Google Cloud Platform(GCP) or Amazon Web Services(AWS). Using these services require paid subscriptions which may be costly for learning developers.

## Installation

### Prerequisites

- kubectl
- docker (for Mac)
- minikube
- virtualbox

We will go through the following steps for macOS platform to setup our environment:

- Install Kubernetes command-line tool,`kubectl`:

```sh
brew install kubectl
```

Minikube can run either on the host or inside a Virtual Machine(VM). Use of a VM is the recommended way and in this tutorial we will be using VirtualBox.

- Download & install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) for OS X. Minikube relies on these drivers.
- Install Minikube:

```sh
brew cask install minikube
```

We can install all the above requirements with a single command:

```sh
brew update && brew install kubectl && brew cask install docker minikube virtualbox
```

To explore setting up Kubernetes and Minikube for other platforms, follow the links:

- [Kubernetes](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [Minikube](https://github.com/kubernetes/minikube)

# Kubernetes Objects

Kubernetes contains a number of abstractions that represent the state of your system: deployed containerized applications and workloads, their associated network and disk resources, and other information about what your cluster is doing. These abstractions are represented by objects in the Kubernetes API.

- [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) - A Pod represents a unit of deployment: a single instance of an application in Kubernetes, which might consist of either a single container or a small number of containers that are tightly coupled and that share resources. Docker is the most common container runtime used in a Kubernetes Pod, but Pods support other container runtimes as well. Some principles around pods are:
  - Containers always run inside a Pod.
  - A pod usually has 1 container but can have more.
  - Containers in the same pod are guaranteed to be located on the same machine and share resources.

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) - A Deployment is as an entity that manages Pods, either a single Pod or a replica set (set of instances of the same Pod configuration). Say for instance, your application has two microservices: a web server and an api server. You will need to define a Pod and Deployment for each microservice. In the Pod you will define which image to run and in the Deployment you will define how many instances of this Pod you will want to have at any given point (for scalability, load balancing, etc…). The Deployment object will then:

  - Create the pod (or multiple instance of it) for your microservice.
  - Monitor these pods to make sure they are up and and healthy. If one fails, the Deployment object will create a new pod to take it’s place.

  After the creation of the Deployment object. You can then:

  - Scale up or down your pods.
  - Rollout new changes to the pods.
  - Rollback changes from pods.

- [Service](https://kubernetes.io/docs/concepts/services-networking/service/) - A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them. Services allow you to expose your microservices (that run in pods) both internally and externally between the other running pods. The service object exposes a consistent IP and port for the microservice and any call to that IP will be rouited to one of the microservice’s running Pods.

The benefit of services is that while each and every pod is assigned an IP, Pods can be shut down (due to failure or rollout of new changes) and the pod’s ip will no longer be available. Meanwhile the service IP and port will not change and always accessible to the other pods in your cluster. By default, the DNS service of Kubernetes exposes the service name to all pods so they can refer to it simply by its name.

- [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) - A job creates one or more pods and ensures that a specified number of them successfully terminate. As pods successfully complete, the job tracks the successful completions. When a specified number of successful completions is reached, the job itself is complete. Deleting a Job will cleanup the pods it created.

- [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) - A ReplicaSet ensures that a specified number of pod replicas are running at any given time.

## [Describing a Kubernetes Object](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)

Kubernetes objects are represented in `.yaml` format. In the `.yaml` file for the Kubernetes object you want to create, you’ll need to set values for the following fields:

- apiVersion - Which version of the Kubernetes API you’re using to create this object
- kind - What kind of object you want to create
- metadata - Data that helps uniquely identify the object, including a name string, UID, and optional namespace

You’ll also need to provide the object `spec` field. The precise format of the object spec is different for every Kubernetes object, and contains nested fields specific to that object. Here's a sample definition for a Deployment object.

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

In the above example:

- A Deployment named `nginx-deployment` is created, indicated by the metadata: name field.
- The Deployment creates three replicated Pods, indicated by the replicas field.
- The selector field defines how the Deployment finds which Pods to manage. In this case, we simply select on one label defined in the Pod template (app: nginx). 
- The Pod template: spec field, indicates that the Pods run one container, nginx, which runs the nginx Docker Hub image at version 1.7.9.
- The Deployment opens port 80 for use by the Pods so that the container can send and accept traffic.

To create this Deployment, run the following command:

```sh
kubectl create -f https://raw.githubusercontent.com/kubernetes/website/master/docs/concepts/workloads/controllers/nginx-deployment.yaml
```

We can do a simple Deployment with the above configuration with the following steps.

```sh
minikube start #Start minikube
```


~IMAGE: minikube-start~

We can then test if minikube is working successfully.

```sh
kubectl get nodes
```

~IMAGE: check-k8s~

To work directly with the Docker deamon on the minikube cluster run:

```sh
eval $(minikube docker-env)
```