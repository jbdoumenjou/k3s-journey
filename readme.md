# How to test Traefik with k3d


## Context

Working with kubernetes could be difficult and heavy for a developer (or a lambda person), that's why some tools like [k3s](https://github.com/rancher/k3s/) are useful to work with a "lightweight" Kubernetes environment.
[K3d](https://github.com/rancher/k3d) is an helper to launch k3s with docker.

This repository contains several scenario, each directory is a step from beginner to advance usage of k3d & Traefik.
The goal is not to provide production ready configuration, this guide is for helping user/developer to understand
how Traefik works and how to test it easily with Kubernetes.
 
## First step

The [step1](step1) directory presents the very first step in the k3d usage with Traefik.
You will see how to start k3d and start our own Traefik. 

# Command Cheat Sheet

```bash
# launch the cluster, exports the wanted ports and avoid that the default traefik configuration to be deployed
k3d create --api-port 6550 --publish 80:80 --publish 443:443 --publish 8080:8080 --server-arg '--no-deploy=traefik' 

# set your environement variable to access the cluster with kubectl
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"

# import a freshly compiled image of traefik into the cluster
k3 import-images containous/traefik:latest 

# check the stack
kubectl get all --all-namespaces

# apply the yaml configuration in the conf directory
kubectl apply -f conf/

# kill the cluster
k3d delete
```

# References

## Glossary

* Helm Charts: A package manager for Kubernetes

## Links
* [k3d](https://github.com/rancher/k3d)
* [k3s](https://github.com/rancher/k3s/)
* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [kubectl Overview](https://kubernetes.io/docs/reference/kubectl/overview/)
* [Helm Charts](https://helm.sh/)
* Kubernetes Notions
  * [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
  * [Service](https://kubernetes.io/fr/docs/concepts/services-networking/service/)
* [Traefik v2 k8s CRD](https://docs.traefik.io/v2.2/providers/kubernetes-crd/)
* [Traefik v2 k8s CRD Routing](https://docs.traefik.io/v2.2/routing/providers/kubernetes-crd/)
* [Traefik v2 k8s CRD Reference](https://docs.traefik.io/v2.2/reference/dynamic-configuration/kubernetes-crd/)
* [Traefik v2 k8s Ingress](https://docs.traefik.io/v2.2/providers/kubernetes-ingress/)
* [Traefik v2 k8s Ingress Routing](https://docs.traefik.io/v2.2/routing/providers/kubernetes-ingress/)
* [Traefik Helm Chart](https://github.com/containous/traefik-helm-chart)
