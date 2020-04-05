# How to test Traefik with k3d


## Context

Working with kubernetes could be difficult and heavy for a developer (or a lambda person), that's why some tools like [k3s](https://github.com/rancher/k3s/) are useful to work with a "lightweight" Kubernetes environment.
[K3d](https://github.com/rancher/k3d) is an helper to launch k3s with docker.


## First step

### prerequisites

* Have the basics in docker and kubernetes
* Installed k3d by following the [instructions](https://github.com/rancher/k3d#get) from the project page.


### First launch

```bash
k3d create 
```

```bash
INFO[0000] Created cluster network with ID dd812c947977dc2956da7fe23911444a9c3b9c2c135fe659cfe38ce2e0846709 
INFO[0000] Created docker volume  k3d-k3s-default-images 
INFO[0000] Creating cluster [k3s-default]               
INFO[0000] Creating server using docker.io/rancher/k3s:v1.0.1... 
INFO[0000] Pulling image docker.io/rancher/k3s:v1.0.1... 
INFO[0025] SUCCESS: created cluster [k3s-default]       
INFO[0025] You can now use the cluster with:

export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
kubectl cluster-info
```

Then we have to set the KUBECONFIG environment variable to use the `kubectl`command and access the cluster

```bash
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
```

```bash
kubectl cluster-info
```

```bash
Kubernetes master is running at https://localhost:6443
CoreDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://localhost:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'
```

Then, we can check the deployment use by default

```bash
kubectl get all --all-namespaces
```

```bash
NAMESPACE     NAME                                          READY   STATUS      RESTARTS   AGE
kube-system   pod/local-path-provisioner-58fb86bdfd-tr5p2   1/1     Running     0          7m40s
kube-system   pod/metrics-server-6d684c7b5-9698x            1/1     Running     0          7m40s
kube-system   pod/helm-install-traefik-pjvdl                0/1     Completed   0          7m41s
kube-system   pod/coredns-d798c9dd-l58mq                    1/1     Running     0          7m40s
kube-system   pod/svclb-traefik-7srdk                       3/3     Running     0          7m1s
kube-system   pod/traefik-65bccdc4bd-xmk8d                  1/1     Running     0          7m1s

NAMESPACE     NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                     AGE
default       service/kubernetes       ClusterIP      10.43.0.1       <none>        443/TCP                                     7m56s
kube-system   service/kube-dns         ClusterIP      10.43.0.10      <none>        53/UDP,53/TCP,9153/TCP                      7m55s
kube-system   service/metrics-server   ClusterIP      10.43.166.22    <none>        443/TCP                                     7m52s
kube-system   service/traefik          LoadBalancer   10.43.119.177   172.18.0.2    80:31919/TCP,443:31008/TCP,8080:31661/TCP   7m1s

NAMESPACE     NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/svclb-traefik   1         1         1       1            1           <none>          7m1s

NAMESPACE     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/local-path-provisioner   1/1     1            1           7m54s
kube-system   deployment.apps/metrics-server           1/1     1            1           7m52s
kube-system   deployment.apps/coredns                  1/1     1            1           7m55s
kube-system   deployment.apps/traefik                  1/1     1            1           7m1s

NAMESPACE     NAME                                                DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/local-path-provisioner-58fb86bdfd   1         1         1       7m40s
kube-system   replicaset.apps/metrics-server-6d684c7b5            1         1         1       7m40s
kube-system   replicaset.apps/coredns-d798c9dd                    1         1         1       7m40s
kube-system   replicaset.apps/traefik-65bccdc4bd                  1         1         1       7m1s

NAMESPACE     NAME                             COMPLETIONS   DURATION   AGE
kube-system   job.batch/helm-install-traefik   1/1           41s        7m52s
```

Tips: if you have a `watch` command installed, keep an eye on your cluster with a

```bash
watch kubectl get all --all-namespace
```

By default, Traefik will be deployed as a default LoadBalancer service.

To check the Traefik installation, we can check the pod logs

```bash

```

# In a nutshell

The command to know
```bash
# launch the cluster, exports the wanted ports and avoid that the default traefik configuration to be deployed
k3d create --api-port 6550 --publish 80:80 --publish 443:443 --publish 8080:8080 --server-arg '--no-deploy=traefik' 

# set your environement variable to access the cluster with kubectl
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"

# import a freshly compiled image of traefik into the cluster
k3 import-images containous/traefik:latest 

# kill the cluster
k3d delete
```

# References

* [K3d](https://github.com/rancher/k3d)
* [k3s](https://github.com/rancher/k3s/)
* [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [kubectl Overview](https://kubernetes.io/docs/reference/kubectl/overview/)
