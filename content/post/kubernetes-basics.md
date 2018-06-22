---
date: 2018-06-21
title: Kubernets basics
---

## Background & why k8s

### Container
Container has become poplular in recent years. There are a few container standard, like Docker, [rkt](https://coreos.com/rkt/). The advantages by using container technology:
* clean and consistent execution environment across local, staging and production.
* resource isolation between apps.

### Cluster
With sofiscated containers apps in use, we need a way to scale and manage the containers, then container orchestrator comes out, like:
* kubernetes
* docker swarm

> Kubernetes is a Container Orchestrator, that abstracts the underlying infrastructure (where the container are run).

K8s通过提供易用的API对底层的架构进行了抽象，调用对应的API请求，就可以完成复杂的基于容器的服务编排和管理。

Why use k8s such thing:
* standardized the Cloud Service Providers, like AWS,GCP,etc.
* manage resources as a whole, reduce cost

## Elements in k8s
* API server: the way to interact with k8s cluster.
* Kubelet: monitor containers in a node, communicate with master node.
* Pods: Pods can be composed of one or a group of containers that share the same execution environment.

![Pod properties]()
Features of pods:
* Each pod has a unique IP address in the k8s cluster.
* Pod can have multiple containers
* Containers in the same pod share volume, ip, port space, IPC namespace.

## Practice
### [Install minikube and kubectl](https://kubernetes.io/docs/tasks/tools/install-minikube/)

Minikube is a tool to run k8s locally.

Install kubectl on macOS:
```
brew install kubectl
```
Check the installation by using `kubectl version`

Install minikube:
```
brew cask install minikube
```
Start cluster in local by using `minikube start` 
If you occurs error in start minikube, try
```
minikube delete
rm -rf ~/.minikube
```

### Pod definition
```
apiVersion: v1
kind: Pod                                            
metadata:
  name: hello-world                                 
spec:                                                
  containers:
    - image: rinormaloku/sentiment-analysis-frontend 
      name: sa-frontend                              
      ports:
        - containerPort: 80   
```

## References
1. https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882

