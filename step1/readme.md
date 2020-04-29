# Step 1 - my first stack with k3d

## Goal

Launch k3d with Traefik v2 

## Prerequisites

* Have the basics in docker and kubernetes
* Installed k3d by following the [instructions](https://github.com/rancher/k3d#get) from the project page.

## Start k3d "As Is“

```bash
$k3d create 
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
$kubectl cluster-info
Kubernetes master is running at https://localhost:6443
CoreDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://localhost:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'
```

Then, we can check the deployment use by default

```bash
$kubectl get all --all-namespaces
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

Tip: if you have a `watch` command installed, keep an eye on your cluster by using it.

```bash
watch kubectl get all --all-namespace
```

By default, Traefik will be deployed as a default LoadBalancer service thanks to a [helm](https://github.com/containous/traefik-helm-chart) installation.
[Helm Charts](https://helm.sh/) is a package manager for Kubernetes. You can find the last version of Traefik Helm Chart in the [official repository]() 

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

## My very first Traefik deployment

As we want to manage our own Traefik, we will apply our own configuration (all the configuration is available in the [conf-0/](conf-0/) directory).
To do that, we need to define a bunch of yaml configuration files (we will not use helm-chart here)

First, we need to use a basic [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) to allow Traefik
to be used as a loadBalancer [Service](https://kubernetes.io/fr/docs/concepts/services-networking/service/).

```yaml
# RBAC
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: traefik-ingress-controller

rules:
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: traefik-ingress-controller

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
  - kind: ServiceAccount
    name: traefik-ingress-controller
    namespace: default

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller

```

Then we will use a [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) to deploy a Traefik
[Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) and apply the `traefik-ingress-controller` ServiceAccount.
For this step, we will only activate the dashboard by using [insecure mode](https://docs.traefik.io/operations/api/#insecure) (inappropriate for the production)

```yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: traefik
  labels:
    app: traefik

spec:
  replicas: 1
  selector:
    matchLabels:
      app: traefik
  template:
    metadata:
      labels:
        app: traefik
    spec:
      serviceAccountName: traefik-ingress-controller
      containers:
        - name: traefik
          image: traefik:v2.2
          imagePullPolicy: Always
          args:
            - --log.level=DEBUG
            - --api.insecure
          ports:
            - name: api
              containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: traefik
spec:
  selector:
    app: traefik
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      name: api
  type: LoadBalancer
```

Let's test, we will launch the k3d with the 8080 HTTP port exposed.
To avoid launching the default Traefik, we have to specify a k3s server option `--no-deploy=traefik`.

```bash
k3d create --publish 8080:8080 --server-arg '--no-deploy=traefik' 
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
watch kubectl get all --all-namespaces
```

Wait for the cluster to be completely launched. Then apply the configuration:

```bash
kubectl apply -f conf-0/
```

As we didn't specify any namespace, the 'default' one will be used.
Let's check the stack (focused on the default namespace).

```bash
$kubectl get all
NAME                          READY   STATUS    RESTARTS   AGE
pod/svclb-traefik-clw9v       1/1     Running   0          2m2s
pod/traefik-6f7c5d67c-p2fkg   1/1     Running   0          2m3s


NAME                 TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)          AGE
service/kubernetes   ClusterIP      10.43.0.1    <none>         443/TCP          2m46s
service/traefik      LoadBalancer   10.43.1.68   192.168.48.2   8080:30069/TCP   2m3s

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/svclb-traefik   1         1         1       1            1           <none>          2m2s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/traefik   1/1     1            1           2m3s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/traefik-6f7c5d67c   1         1         1       2m3s
```

We can open a browser and check the [dashboard](http://localhost:8080/dashboard)

The cluster can be stopped

```bash
k3d d
```

## Who Am I ?

Now that we know how to launch Traefik, let's try to route the traffic to our wonderful whoami!
All the configuration is available under the [conf-1](./conf-1) directory.

As we want to route the traffic to some instances of whoami, we will use a 
[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) as for Traefik instance deployment.
The only notable difference is the replicas attributes that will be set to have several whoami instances:

````yaml
# whoami.yml
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: whoami
  namespace: default
  labels:
    app: containous
    name: whoami

spec:
  replicas: 2
  selector:
    matchLabels:
      app: containous
      task: whoami
  template:
    metadata:
      labels:
        app: containous
        task: whoami
    spec:
      containers:
        - name: containouswhoami
          image: containous/whoami
          ports:
            - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: whoami
  namespace: default

spec:
  ports:
    - protocol: TCP
      port: 8080
  selector:
    app: containous
    task: whoami
````

Traefik service need to be updated to define how to route the traffic to them.
There are several ways to configure Traefik, in this example we will use the "CRD" one.
Kubernetes provide a collection of resources (like services, pods, etc...) it could be extended thanks to Custom Resource Definitions ([CRD](TO COMPLETE)).
The first thing is to declare this new definitions:

```yaml
# definitions.yml
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressroutes.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRoute
    plural: ingressroutes
    singular: ingressroute
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: middlewares.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: Middleware
    plural: middlewares
    singular: middleware
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressroutetcps.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRouteTCP
    plural: ingressroutetcps
    singular: ingressroutetcp
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: ingressrouteudps.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: IngressRouteUDP
    plural: ingressrouteudps
    singular: ingressrouteudp
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: tlsoptions.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: TLSOption
    plural: tlsoptions
    singular: tlsoption
  scope: Namespaced

---
apiVersion: apiextensions.k8s.io/v1beta1
kind: CustomResourceDefinition
metadata:
  name: traefikservices.traefik.containo.us

spec:
  group: traefik.containo.us
  version: v1alpha1
  names:
    kind: TraefikService
    plural: traefikservices
    singular: traefikservice
  scope: Namespaced
```

Traefik will need all the definitions to check the configuration, 
that's means that all the definitions must be declared in your cluster,
 may they be used or not in your configuration. (TODO: rephrase)
 
Now that the definitions are exposed, Traefik needs the authorization to use them, the RBAC must be updated accordingly.

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: traefik-ingress-controller

rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses/status
    verbs:
      - update
# the following part allows Traefik to use the new resources
  - apiGroups:
      - traefik.containo.us
    resources:
      - middlewares
      - ingressroutes
      - traefikservices
      - ingressroutetcps
      - ingressrouteudps
      - tlsoptions
    verbs:
      - get
      - list
      - watch

---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: traefik-ingress-controller

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
  - kind: ServiceAccount
    name: traefik-ingress-controller
```

Declaring resources definition is one thing, having an instance of a resource is another thing.
In our case, Traefik have to route HTTP requests to the whoami service, the [IngressRoute](TODO add lik to the doc) resource is the answer.
Let's see the IngressRoute definition:

```yaml
# ingressroute.yml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: myingressroute
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`mydomain`)
      kind: Rule
      services:
        - name: whoami
          port: 8080
```

We create an instance of `IngressRoute` resource with the apiVersion `traefik.containo.us/v1alpha1`.
All the request with the `mydomain` Host will be routed to the `whoami` kubernetes service on the `8080` port.

Last but not least, Traefik must be configured to listen to kubernetes and use the IngressRoute resource.
So, Traefik needs to defined:
 * the web HTTP entrypoint 
 * the KubernetesCRD provider

To allow request to access Traefik, the ports must be open:
* in the Traefik deployment
* in the ingress service
* in the kubernetes cluster

Let's see the Traefik configuration:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller

---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: traefik
  labels:
    app: traefik

spec:
  replicas: 1
  selector:
    matchLabels:
      app: traefik
  template:
    metadata:
      labels:
        app: traefik
    spec:
      serviceAccountName: traefik-ingress-controller
      containers:
        - name: traefik
          image: traefik:v2.2
          args:
            - --log.level=DEBUG
            - --api.insecure
            - --entrypoints.web.address=:80
            - --providers.kubernetescrd
          ports:
            - name: web
              containerPort: 80
            - name: api
              containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: traefik
spec:
  selector:
    app: traefik
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      name: api
    - protocol: TCP
      port: 80
      targetPort: 80
      name: web
  type: LoadBalancer
```

Now that we have all the configuration, let's try it!
First, we have to launch the cluster with the new port open

```bash
k3d create --publish 8080:8080 --publish 80:80 --server-arg '--no-deploy=traefik' 
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
watch kubectl get all --all-namespaces
```

Then, apply the configuration:

```bash
kubectl apply -f conf-1/
```

And see what's happens:

TODO: update
```bash
NAMESPACE     NAME                                          READY   STATUS              RESTARTS   AGE
default       pod/whoami-bf78b74c6-d5w5t                    0/1     ContainerCreating   0          3m2s
default       pod/whoami-bf78b74c6-9x9lx                    0/1     ContainerCreating   0          3m2s
default       pod/svclb-traefik-7pmwl                       0/2     ContainerCreating   0          3m2s
default       pod/traefik-5c664675bf-cfdj8                  0/1     ContainerCreating   0          3m2s


NAMESPACE     NAME                     TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                       AGE
default       service/kubernetes       ClusterIP      10.43.0.1       <none>        443/TCP                       3m24s
default       service/traefik          LoadBalancer   10.43.150.180   <pending>     8080:31508/TCP,80:31167/TCP   3m3s
default       service/whoami           ClusterIP      10.43.46.130    <none>        8080/TCP                      3m2s

NAMESPACE   NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
default     daemonset.apps/svclb-traefik   1         1         0       1            0           <none>          3m2s

NAMESPACE     NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/whoami                   0/2     2            0           3m2s
default       deployment.apps/traefik                  0/1     1            0           3m3s
```

We have:
 * 2 whoami pods thanks to the replicas attribute configuration.
 * a kubernetes service to expose whoami pods
 * a kubernetes service to expose traefik pod
 * the deployments for Traefik and whoami pods
 
 In this view, we can't see the ingressroute configuration, but we can check it as it is a valid kubernetes resource:
```bash
$ kubectl get ingressroute
NAME             AGE
myingressroute   67s
```

We can see that we have one ingressroute named `myingressroute`.
Let's try to have more information:
```bash
$ kubectl describe ingressroute/myingressroute
Name:         myingressroute
Namespace:    default
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"traefik.containo.us/v1alpha1","kind":"IngressRoute","metadata":{"annotations":{},"name":"myingressroute","namespace":"defau...
API Version:  traefik.containo.us/v1alpha1
Kind:         IngressRoute
Metadata:
  Creation Timestamp:  2020-04-29T10:20:05Z
  Generation:          1
  Resource Version:    636
  Self Link:           /apis/traefik.containo.us/v1alpha1/namespaces/default/ingressroutes/myingressroute
  UID:                 64d53435-5b73-4bb6-b218-a97a4f5a4e9c
Spec:
  Entry Points:
    web
  Routes:
    Kind:   Rule
    Match:  Host(`mydomain`)
    Services:
      Name:  whoami
      Port:  8080
Events:      <none>
```

We can have some useful information like the last applied configuration, the metadata associated to the resource and its definition.
Another way to have information is to use the following command:

````bash
$ kubectl get ingressroute/myingressroute -o yaml
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"traefik.containo.us/v1alpha1","kind":"IngressRoute","metadata":{"annotations":{},"name":"myingressroute","namespace":"default"},"spec":{"entryPoints":["web"],"routes":[{"kind":"Rule","match":"Host(`mydomain`)","services":[{"name":"whoami","port":8080}]}]}}
  creationTimestamp: "2020-04-29T10:20:05Z"
  generation: 1
  name: myingressroute
  namespace: default
  resourceVersion: "636"
  selfLink: /apis/traefik.containo.us/v1alpha1/namespaces/default/ingressroutes/myingressroute
  uid: 64d53435-5b73-4bb6-b218-a97a4f5a4e9c
spec:
  entryPoints:
  - web
  routes:
  - kind: Rule
    match: Host(`mydomain`)
    services:
    - name: whoami
      port: 8080
````
 