# Step 1 - my first stack with k3d

## Goal

Launch k3d with Traefik v2 

## Prerequisites

* Have the basics in docker and kubernetes
* Installed k3d by following the [instructions](https://github.com/rancher/k3d#get) from the project page.


## Start k3d "As Is“

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

Tips: if you have a `watch` command installed, keep an eye on your cluster by using it.

```bash
watch kubectl get all --all-namespace
```

By default, Traefik will be deployed as a default LoadBalancer service thanks to a [helm](https://github.com/containous/traefik-helm-chart) installation.
[Helm Charts](https://helm.sh/) is a package manager for Kubernetes.

To check the Traefik installation, we can check the Traefik pod logs

```bash
kubectl logs pod/traefik-65bccdc4bd-xmk8d -n kube-system
```

Thanks to the log, we can have the version of Traefik (1.7.14 in this case) and the configuration like the EntryPoints exposed.

```bash
{"level":"info","msg":"Using TOML configuration file /config/traefik.toml","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"No tls.defaultCertificate given for https: using the first item in tls.certificates as a fallback.","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Traefik version v1.7.14 built on 2019-08-14_09:46:58AM","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"\nStats collection is disabled.\nHelp us improve Traefik by turning this feature on :)\nMore details on: https://docs.traefik.io/basics/#collected-data\n","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Preparing server http \u0026{Address::80 TLS:\u003cnil\u003e Redirect:\u003cnil\u003e Auth:\u003cnil\u003e WhitelistSourceRange:[] WhiteList:\u003cnil\u003e Compress:true ProxyProtocol:\u003cnil\u003e ForwardedHeaders:0xc000776120} with readTimeout=0s writeTimeout=0s idleTimeout=3m0s","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Preparing server https \u0026{Address::443 TLS:0xc0005d8360 Redirect:\u003cnil\u003e Auth:\u003cnil\u003e WhitelistSourceRange:[] WhiteList:\u003cnil\u003e Compress:true ProxyProtocol:\u003cnil\u003e ForwardedHeaders:0xc000776160} with readTimeout=0s writeTimeout=0s idleTimeout=3m0s","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Starting server on :80","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Preparing server traefik \u0026{Address::8080 TLS:\u003cnil\u003e Redirect:\u003cnil\u003e Auth:\u003cnil\u003e WhitelistSourceRange:[] WhiteList:\u003cnil\u003e Compress:false ProxyProtocol:\u003cnil\u003e ForwardedHeaders:0xc0007761a0} with readTimeout=0s writeTimeout=0s idleTimeout=3m0s","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Starting server on :443","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Starting provider configuration.ProviderAggregator {}","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Starting server on :8080","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Starting provider *kubernetes.Provider {\"Watch\":true,\"Filename\":\"\",\"Constraints\":[],\"Trace\":false,\"TemplateVersion\":0,\"DebugLogGeneratedTemplate\":false,\"Endpoint\":\"\",\"Token\":\"\",\"CertAuthFilePath\":\"\",\"DisablePassHostHeaders\":false,\"EnablePassTLSCert\":false,\"Namespaces\":null,\"LabelSelector\":\"\",\"IngressClass\":\"\",\"IngressEndpoint\":{\"IP\":\"\",\"Hostname\":\"\",\"PublishedService\":\"kube-system/traefik\"}}","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"ingress label selector is: \"\"","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Creating in-cluster Provider client","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Server configuration reloaded on :80","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Server configuration reloaded on :443","time":"2020-04-06T06:59:13Z"}
{"level":"info","msg":"Server configuration reloaded on :8080","time":"2020-04-06T06:59:13Z"}
```

As we want to manage our own Traefik, we will apply our configuration.
To do that, we need to define a bunch of yaml configuration file (we will not use helm-chart here)

We will use a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) to deploy a Traefik
[Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) and a LoadBalancer 
[Service](https://kubernetes.io/fr/docs/concepts/services-networking/service/) to deploy Traefik.
