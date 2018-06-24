---
date: 2018-06-21
title: Kubernetes basics
---

- [Background & why k8s](#background-why-k8s)
	- [Container](#container)
	- [Cluster](#cluster)
- [Elements in k8s](#elements-in-k8s)
- [Practice](#practice)
		- [Install kubectl](#install-kubectl)
		- [Use docker for Mac edge](#use-docker-for-mac-edge)
		- [Use minikube](#use-minikube)
	- [Common used command](#common-used-command)
	- [Access the dashboard](#access-the-dashboard)
	- [Run sample container](#run-sample-container)
	- [Pod definition](#pod-definition)
- [References](#references)


## Background & why k8s

### Container
Container has become poplular in recent years. There are a few container standard, like Docker, [rkt](https://coreos.com/rkt/). The advantages by using container technology:  

* Clean and consistent execution environment across local, staging and production.    
* Resource isolation between apps.
* Born for mircroservices application design pattern.     

### Cluster
With sofiscated containers apps in use, we need a way to scale and manage the containers, then container orchestrator comes out, like:  

* kubernetes    
* docker swarm  

> Kubernetes is a Container Orchestrator, that abstracts the underlying infrastructure (where the container are run).

K8s通过提供易用的API对底层的架构进行了抽象，调用对应的API请求，就可以完成复杂的基于容器的服务编排和管理。

Why use k8s such thing:
* Standardized the Cloud Service Providers, like AWS,GCP,etc.
* Manage resources as a whole, reduce costs.

## Elements in k8s
* API server: the way to interact with k8s cluster.
* Kubelet: monitor containers in a node, communicate with master node.

### Pods
Pods can be composed of one or a group of containers that share the same execution environment.

![Pod properties]()

Features of pods:     

* Each pod has a unique IP address in the k8s cluster.     
* Pod can have multiple containers    
* Containers in the same pod share volume, ip, port space, IPC namespace.   
 
### Deployments


### Services

## Practice
### Install kubectl
Install kubectl on macOS:
```
brew install kubectl
```
Check the installation by using `kubectl version`

### Use docker for Mac edge
The edge version of docker for mac, provides k8s integration, you need to [download](https://store.docker.com/editions/community/docker-ce-desktop-mac) it, and enable the k8s feature.

![enable-k8s-edge](/static/k8s/enable-k8s-edge.png)

Check kubectl are using docker-for-desktop:
```
kubectl config current-context
```
If not, switch to it `kubectl config use-context docker-for-desktop`.

If you are using docker edge to play with k8s, you could skip the section about `minikube`.

### Use minikube

[Minikube]((https://kubernetes.io/docs/tasks/tools/install-minikube/)) is a tool to run k8s locally. I have bad experience when using minikube.

Install minikube:
```
brew cask install minikube
```
Start cluster in local by using `minikube start`, or using `minikube start --bootstrapper=localkube` if there is error happens when using kubeadm bootstrapper.

If you occurs error in start minikube, try
```
minikube delete
rm -rf ~/.minikube
```

### Common used commands    
* `kubectl cluster-info`    
* `kubectl get pods`    
* `kubectl get nodes`   

### Access the dashboard
First, deploy the dashboard since it's not deployed by default:
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
Show all the services `kubectl get pods --all-namespaces`, you should be able see `kubernetes-dashboard` is `running`.

Forward to the dashboard pod:
```
kubectl port-forward kubernetes-dashboard-7d5dcdb6d9-nkrx5 8443:8443 --namespace=kube-system
```

Access [https://localhost:8443](https://localhost:8443), you will see the dashboard UI.
![dashboard](/static/k8s/Kubernetes_Dashboard.png)

### Run sample container
In this section, we are going to setup Nginx container for test purpose.

Create a deployment about the container:    
```
kubectl run hello-nginx --image=nginx --port=80
```
Expose the basic Nginx deployment as a service:
```
kubectl get deployments

kubectl expose deployment hello-nginx --type=NodePort
```
You should be able to see `Service "hello nginx" exposed"`    
If you switch to the dashboard, you should be able to see the added service in the Services section. Also you could use `kubectl get services` and `kubectl describe service hello-nginx`.    
Want to access the Nginx? Remember to forward the port first.   
```
kubectl get pods # show all the pods
kubectl port-forward hello-nginx-6f9f4fc7dd-6wzlp 8080:80 # forward one of the pods
```

Scala the service:
```
kubectl scale --replicas=3 deployment/hello-nginx
```
Go to Discovery and load balancing -> Services -> hello-nginx in dashboard, there should be 3 pods.

Clean up:
```
kubectl delete service hello-nginx
kubectl delete deployment hello-nginx
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
2. Serverless Kubernetes with OpenFaaS: https://github.com/openfaas/faas-netes
3. Setup in local: https://gist.github.com/kevin-smets/b91a34cea662d0c523968472a81788f7
4. Survive from GFW: https://github.com/denverdino/k8s-for-docker-desktop
5. K8s with docker edge: https://rominirani.com/tutorial-getting-started-with-kubernetes-with-docker-on-mac-7f58467203fd
