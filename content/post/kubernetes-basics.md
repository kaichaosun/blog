---
date: 2018-06-21
title: Kubernetes basics
---

## Background & why kubernetes

### Micro services
Usually we build a single "monolith" application at first time. As the business grows, the app becomes "heavy" and hard to maintain:  

* Improved cost to communicate within different teams;
* Long enough time to finish the CI pipeline(run the test cases, packages, deployment);
* Hard to scale;
* ...

So we need to make big application to be small different services depends on a few principles, that's micro services. But it's not the silver bullet. Since there is never such thing in software development. With more and more micro services:

* Take lots of cost to host all the services;
* Don't have a standard way to implement the CI/CD;

### Container

Now docker and other container technology comes into our vision. Container has become popular in recent years. There are a few container standard, like Docker, [rkt](https://coreos.com/rkt/). The advantages by using container technology:  

* Clean and consistent execution environment across local, staging and production.    
* Resource isolation between apps.
* Born for micro services application design pattern.     

Now we have solved the above questions by leveraging docker. What other problems we got?

* Too many pipelines for each container app;   
* Containerized app in same VM don't work as well as expected;   
* Costs still not reduced   

### Cluster

With complex containers apps in use, we need a way to scale and manage the containers, then container orchestrator comes out, like:  

* kubernetes    
* docker swarm  

> Kubernetes is a Container Orchestrator, that abstracts the underlying infrastructure (where the container are run).

K8s通过提供易用的API对底层的架构进行了抽象，调用对应的API请求，就可以完成复杂的基于容器的服务编排和管理。

Why use k8s such thing:
* Standardized the Cloud Service Providers, like AWS, GCP, etc.
* Manage resources as a whole, reduce costs.

## Architecture in kubernetes
![k8s-architecture](/static/k8s-architecture.png)

### Master node
The master node is responsible for the management of the cluster. It response to the admin operations, reflect the configuration in certain worker node, ensure the cluster state is as expected. Master node contains following components:

* API server: the way to interact with k8s cluster through REST requests.
* etcd: key-value store, used for all cluster data.
* scheduler: watch newly created pods, select a node for them.
* controller-manager: runs controllers (one of concepts in k8s)
* cloud-controller-manager: runs controllers that interact with the underlying cloud providers. Details refer [here](https://kubernetes.io/docs/concepts/overview/components/)



### Worker node

There are a few components running on every worker node.

* kubelet: makes sure that the containers are running as expected in a worker node.
* kube-proxy: enables the k8s service abstraction by maintaining network rules on the host and performing connection forwarding.
* Container Runtime: responsible for running containers, like Docker, rkt.


## Useful services

### Pods

Can be composed of one or a group of containers that share the same execution environment. Users should not create pods directly, better to use controllers like `Deployments` for self-healing.

Features of pods:     

* Each pod has a unique IP address in the k8s cluster.     
* Pod can have multiple containers    
* Containers in the same pod share volume, ip, port space, IPC namespace.   

### Controllers

**Deployment:**

Declare how many replicas of a pod should be running at same time. When deployment is applied in the cluster, it will automatically spin up the request number of pods. Then monitor them, if a pod dies, the deployment will re-create a new pod to meet the request number.

Define a Deployment:  
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
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
        image: nginx:1.9.1
        ports:
        - containerPort: 80

```

Run command:  
```
kubectl create -f ./nginx-deployment.yml --record

kubectl get deployments

kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1

kubectl describe deployments

kubectl rollout history deployment/nginx-deployment

kubectl rollout undo deployment/nginx-deployment  // optional --to-revision=2

// scale deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl autoscale deployment nginx-deployment --min=3 --max=6 --cpu-percent=80  

kubectl delete deployment nginx-deployment

```

**Job:**

Run one off tasks. A Job creates Pods that run until successful termination (i.e., exit with 0)

**ReplicaSet:**


### Services
K8s services is an abstraction which defines a set of pods and how to access them.

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

## Using with GCP and AWS
Google cloud provide GKE.

You can simply start a cluster in AWS with [kops](https://github.com/kubernetes/kops)


## References
* https://medium.freecodecamp.org/learn-kubernetes-in-under-3-hours-a-detailed-guide-to-orchestrating-containers-114ff420e882  
* https://github.com/openfaas/faas-netes  
* https://gist.github.com/kevin-smets/b91a34cea662d0c523968472a81788f7  
* https://github.com/denverdino/k8s-for-docker-desktop  
* https://rominirani.com/tutorial-getting-started-with-kubernetes-with-docker-on-mac-7f58467203fd   
* https://medium.com/google-cloud/kubernetes-110-your-first-deployment-bf123c1d3f8   
* https://github.com/kubernetes-incubator/kubespray/blob/master/docs/comparisons.md  
* https://blog.hasura.io/gke-vs-aks-vs-eks-411f080640dc  
* https://x-team.com/blog/introduction-kubernetes-architecture/  
