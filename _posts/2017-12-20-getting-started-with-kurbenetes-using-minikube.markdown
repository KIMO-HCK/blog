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

- Planet Scale - Kubernetes is designed on the same principles that allow Google to run billions of containers a week, Kubernetes can scale without increasing your ops team.

- Never outgrow - Kubernetes flexibility grows to deliver your applications consistently and easily no matter how complex the need is.

- Run anywhere - Kubernetes is open source giving the freedom to take advantage of on-premises, hybrid, or public cloud infrastructure, letting you to effortlessly move workloads to where it matters to you.

# Kubernetes Features

- Horizontal scaling - Scale your application up and down with a simple command, with a UI, or automatically based on CPU usage.
- Automated rollouts and rollbacks - Kubernetes progressively rolls out changes to your application or its configuration, while monitoring application health to ensure it doesn't kill all your instances at the same time. If something goes wrong, Kubernetes will rollback the change for you.
- Self-healing - Restarts containers that fail, replaces and reschedules containers when nodes die, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.
- Service discovery and load balancing - Kubernetes gives containers their own IP addresses and a single DNS name for a set of containers and can load-balance across them.
- Secret and configuration management - Objects of type `secret` are used to hold sensitive information like passwords, OAuth tokens, and ssh keys. Kubernetes allows us to deploy and update secrets and application configuration without rebuilding our images.

These ate just some of the main features that we get when using Kubernetes.

# What is Minikube

Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a VM on your laptop for users looking to try out Kubernetes or develop with it day-to-day.

Alternatives to using Minikube include deploying applications to Google Cloud Platform(GCP) or Amazon Web Services(AWS). Using these services require paid subscriptions which may be costly for learning developers.

## Installation

### Prerequisites

- kubectl(Kubernetes command-line tool)
- Docker (for Mac)
- Minikube
- Virtualbox

We will go through the following steps for macOS platform to setup our environment:

- Install Kubernetes command-line tool,`kubectl`:

```sh
$ brew install kubectl
```

Minikube can run either on the host or inside a Virtual Machine(VM). Use of a VM is the recommended way and in this tutorial, we will be using VirtualBox.

- Download & install [VirtualBox](https://www.virtualbox.org/wiki/Downloads) for OS X. Minikube relies on these drivers.
- Install Minikube:

```sh
$ brew cask install minikube
```

We can install all the above requirements with a single command:

```sh
$ brew update && brew install kubectl && brew cask install docker Minikube virtualbox
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

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) - A Deployment is as an entity that manages Pods, either a single Pod or a replica set (set of instances of the same Pod configuration). Say, for instance, your application has two microservices: a web server and an API server. You will need to define a Pod and Deployment for each microservice. In the Pod you will define which image to run and in the Deployment you will define how many instances of this Pod you will want to have at any given point (for scalability, load balancing, etc…). The Deployment object will then:

  - Create the pod (or multiple instances of it) for your microservice.
  - Monitor these pods to make sure they are up and healthy. If one fails, the Deployment object will create a new pod to take its place.

  After the creation of the Deployment object. You can then:

  - Scale up or down your pods.
  - Rollout new changes to the pods.
  - Rollback changes from pods.

- [Service](https://kubernetes.io/docs/concepts/services-networking/service/) - A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them. Services allow you to expose your microservices (that run in pods) both internally and externally between the other running pods. The service object exposes a consistent IP and port for the microservice and any call to that IP will be routed to one of the microservice’s running Pods.

The benefit of services is that while each and every pod is assigned an IP, Pods can be shut down (due to failure or rollout of new changes) and the pod’s IP will no longer be available. Meanwhile the service's IP and port will not change and always accessible to the other pods in your cluster. By default, the DNS service of Kubernetes exposes the service name to all pods so they can refer to it simply by its name.

- [Job](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/) - A job creates one or more pods and ensures that a specified number of them successfully terminate. As pods successfully complete, the job tracks the successful completions. When a specified number of successful completions is reached, the job itself is complete. Deleting a Job will clean up the pods it created.

- [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) - A ReplicaSet ensures that a specified number of pod replicas are running at any given time.

## [Describing a Kubernetes Object](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)

Kubernetes objects are represented in `.yaml` format. In the `.yaml` file for the Kubernetes object you want to create, you’ll need to set values for the following fields:

- apiVersion - Which version of the Kubernetes API you’re using to create this object
- kind - What kind of object you want to create
- metadata - Data that helps uniquely identify the object, including a name string, UID, and an optional namespace

You’ll also need to provide the object `spec` field. The precise format of the object spec is different for every Kubernetes object and contains nested fields specific to that object. Here's a sample definition of a Deployment object called `nginx-deployment.yaml`.

```yaml
apiVersion: apps/v1beta2 # for Kubernetes versions before 1.9.0 use apps/v1beta2
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

We can do a simple Deployment with the above configuration with the following steps.

```sh
$ minikube start # Start Minikube
```

~IMAGE minikube-start~

The image above shows the version of Kubernetes(1.8.0) running on our server; Minikube is our server. We can also check this by running it with kubectl. The 'Major' and 'Minor' sections under 'Server Version' confirm our server version.

```sh
$ kubectl version
```

~IMAGE kubectl-client-server-versions~

We can now do the actual creation of the deployment.

```sh
$ kubectl get nodes # Optional. Check if Minikube is working properly
$ kubectl create -f nginx-deployment.yaml  # Create the deployment
```

We can then check for available deployments with:

```sh
$ kubectl get deploy
```

~IMAGE get_deployments~

Creation of the above deployment also does some work under the hood depending on the configuration file. We can confirm this by checking the Minikube dashboard. A replica set is automatically created and from the same configuration, 3 pods are created under one deployment. 

```sh
$ minikube dashboard
```

The dashboard has a lot of information that reflect our setup. On the far left is a menu that we can use to check pods, deployments, replica sets, services and a lot more. We will also view this information in our consoles in a shot while using some commands.

~IMAGE dashboard~

# Creating a Real Application

The above sections have been a good introduction to important concepts. In this section, we now focus on implementing it on a real application. We will be using a Node.js application.

```JAVASCRIPT
//server.js
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};
var www = http.createServer(handleRequest);
www.listen(8080);
```

## Creating a Docker container image

We then create a `Dockerfile`. A Dockerfile describes the image that we want to build. We can build a Docker container image by extending an existing image. The image in this tutorial extends an existing Node.js image.

```Dockerfile
FROM mhart/alpine-node:8
EXPOSE 8080
COPY server.js .
CMD node server.js
```

This recipe for the Docker image starts from a minimal Node.js image found in the Docker registry, exposes port 8080, copies your server.js file to the image and starts the Node.js server.

Because this tutorial uses Minikube, instead of pushing our Docker image to a registry, we can simply build the image using the same Docker host as the Minikube VM, so that the images are automatically present. To do so, we use the Minikube Docker daemon.

```sh
$ eval $(minikube docker-env)
```

We can always undo the above change when we are no longer using Minikube using the following command.

```sh
$ eval $(minikube docker-env -u)
```

We can then build a Docker image, using the Minikube Docker daemon. The command basically assigns a name and tag to our resulting image.

> Ensure you are in the directory that has both the Dockerfile and the `server.js` file. The "." at the end of the following command means we are using a Dockerfile from the local directory.

```sh
$ docker build -t hello-node:v1 .
```

In the image below, we can clearly see the different commands from the Dockerfile being executed. You can check out the [Docker reference](https://docs.docker.com/engine/reference/builder/) for more information on Docker commands.

~IMAGE docker-build~

Minikube VM can now run the built image. We can also confirm if the images were built by listing the available images.

```sh
$ docker images
```

~IMAGE docker-images~

# Creating a Deployment

A single Pod is created with only one container. A Kubernetes Deployment checks on the health of your Pod and restarts the Pod’s Container if it terminates. Deployments are the recommended way to manage the creation and scaling of Pods.

`kubectl run` command creates a Deployment that manages a Pod. The Pod runs a Container based on the `hello-node:v1` Docker image. The following command creates a single instance of `hello-node` and lets the container expose port 8080. By exposing we mean, it takes a replication controller, service or pod and exposes it as a new Kubernetes Service; specifically serves on port 8080 and connects to the container on port 8080.

```sh
$ kubectl run hello-node --image=hello-node:v1 --port=8080
```

~IMAGE deployments~

We can check the available deployments to confirm that the above command was successful and we can see hello-node listed.

```sh
$ kubectl get deployments  # Alternatively kubectl get deploy
```

~IMAGE get deployments~

We can also check the pods created using `kubectl get pods`. We can also note from the figure below that we have three pods for the nginx-deployment, this is because we specified 3 replicas earlier on.

~IMAGE pods~

# Creating a Service

By default, the Pod is only accessible by its internal IP address within the Kubernetes cluster. To make the hello-node Container accessible from outside the Kubernetes virtual network, we have to expose the Pod as a Kubernetes Service.

From your development machine, you can expose the Pod to the public internet using the kubectl expose command:

```sh
$ kubectl expose deployment hello-node --type=LoadBalancer
```

~IMAGE expose-service~

We can view the created service with `kubectl get services`. The `--type=LoadBalancer` flag indicates that we want to expose our Service outside of the cluster. On cloud providers that support load balancers, an external IP address would be provisioned to access the Service. On Minikube, the LoadBalancer type makes the Service accessible through the Minikube service command.

```sh
$ minikube service hello-node
```

This will automatically open up a browser window using a local IP address and serve our application by showing "Hello world".

~IMAGE show-browser~

# Updating the Application

We can add some more information to our application, rebuild the Docker image with a new tag and update our deployment to use the new image.

```JAVASCRIPT
//...code before
response.end('Hello World from Kubernetes');
//...code after
```

```sh
$ docker build -t hello-node:v2 .
$ docker images  # List available images
```

Notice the output shows two hello-node images but with different tags.

~IMAGE image-v2~

We can then set the deployment to use the new image and repeat the same process to see our application in the browser.

```sh
$ kubectl set image deployment/hello-node hello-node=hello-node:v2
$ minikube service hello-node
```

~IMAGE browser-v2~

# Points to Note

In the first example with the `nginx-deployment`, we created a configuration file to use but did not create one with hello-node. Kubernetes automatically takes the available configuration file and appends some more information if one is provided. If none is provided, it creates default configurations for Deployments, Pods and Services. Let's explore our dashboard and see the results using `minikube dashboard`.

~IMAGE dashboard-main-2~

Clicking on hello-node under Deployments and then clicking on Edit in the right corner shows us the default configuration.

~ IMAGE hello-node-config~

The best thing about this is we can update the configuration and everything in the cluster will update. Let us update to three replicas for the hello-node deployment and check the number of Pods on our dashboard.

~ IMAGE update-hello-node-dep~

~ IMAGE new-pods ~

We can also do another cool thing. Since we have set `replicas=3` in our configuration, deleting one pod will spin up a new one. This ensures that our services never are always up.

```sh
$ kubectl delete pod-name # Replace the pod name e.g hello-node-657c99955c-dh9sb
```

~ IMAGE respin-pods ~

# Conclusion

Kubernetes is a widely adopted container cluster manager. It allows developers to organize all the different layers of their application through Kubernetes configurations. This requires a developer to understand the different concepts around it through practice. Minikube makes this easy because it provides an almost equivalent to cloud service providers like AWS and GCP hence cutting down the costs of learning.

This article only provides an introduction to the many concepts and benefits that we can get by using Kubernetes to manage our applications. A lot more information can be gained by going through the official documentation and doing more examples.