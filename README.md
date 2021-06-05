# kubernetes-engineer
Knowledge for every k8s engineer

## Kubernetes
### What happens when one types kubectl -f deployment.yaml
good explanation - <https://github.com/jamiehannaford/what-happens-when-k8s>

more pictures - <https://twitter.com/learnk8s/status/1093116707512729601?lang=en-gb>
### Clusters
Information about clusters lifecycle
### Components
Official documentation: <https://kubernetes.io/docs/concepts/overview/components/>

Architecture diagram ![](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg) 

### Kubectl
The Kubernetes command-line tool, kubectl, allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs. For more information including a complete list of kubectl operations, see the kubectl reference documentation.

Kubectl reference: <https://kubernetes.io/docs/reference/kubectl/>
### Deployments
A Deployment provides declarative updates for Pods and ReplicaSets. The deployment creates a ReplicaSet object based on 'replicas' parameter which creates pod object according to the pod template (spec.template).
Example deployment definition:
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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```
It is straight forward to update a deployment. For example to change docker image of the container:

```
kubectl --record deployment.apps/nginx-deployment set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
```
The Deployment updates Pods in a rolling update fashion when .spec.strategy.type==RollingUpdate (which is default). You can specify maxUnavailable and maxSurge to control the rolling update process.

There are ways to provide different deployment strategies - canary, blue/green etc.


### Pods
Pods are the smallest deployable units of computing that you can create and manage in Kubernetes. Pod (as in pod of dolphins) is a group of oe or more containers. All the containers within pod run in shared context, e.g. those can share filesystem.
Alongside application pods one can run init containers (which runs first) or a sidecar containers.
Sample pod definition:
```
apiVersion: v1
kind: Pod
metadata:
  name: cpu-demo
  namespace: cpu-example
spec:
  containers:
  - name: cpu-demo-ctr
    image: vish/stress
    resources:
      limits:
        cpu: "1"
      requests:
        cpu: "0.5"
    args:
    - -cpus
    - "2"
```
### Nodes
See [here](#components)
### Requests and limits
Managing resources for containers: <https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/>

When you specify the resource request for Containers in a Pod, the scheduler uses this information to decide which node to place the Pod on. When you specify a resource limit for a Container, the kubelet enforces those limits so that the running container is not allowed to use more of that resource than the limit you set. The kubelet also reserves at least the request amount of that system resource specifically for that container to use.


There are three types of resources one can define requests and limits for:
* CPU
* memory
* huge pages

### Metrics and logging
### Troubleshooting
### Accessing applications hosted in Kubernetes

### Pod controllers
We can distinguish following most popular pod controllers:

* ReplicaSet - A ReplicaSet creates a stable set of pods, all running the same workload. You will almost never create this directly.
* Deployment - A Deployment is the most common way to get your app on Kubernetes. It maintains a ReplicaSet with the desired configuration, with some additional configuration for managing updates and rollbacks.
* StatefulSet - A StatefulSet is used to manage stateful applications with persistent storage. Pod names are persistent and are retained when rescheduled (app-0, app-1). Storage stays associated with replacement pods, and volumes persist when pods are deleted.
* Job - A Job creates one or more short-lived Pods and expects them to successfully terminate.
* CronJob - A CronJob creates Jobs on a schedule.
* DaemonSet - A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Common for system processes like CNI, Monitor agents, proxies, etc.

### RBAC
Using RBAC: <https://kubernetes.io/docs/reference/access-authn-authz/rbac/>

RBAC is used to provide authentication and authorization on the k8s API server (and the whole cluster essentially).

RBAC can be enabled on the API server by configuring the --authorization-mode to include RBAC:

```
kube-apiserver --authorization-mode=other methods,RBAC
```

The RBAC API declares four kinds of Kubernetes object: Role, ClusterRole, RoleBinding and ClusterRoleBinding:

1. Role and ClusterRole

Role and ClusterRole define set of rules to access resources within the cluster. Permissions are purely additive - there is no deny rule.

Role is defined per namespace and ClusterRole is  defined cluster-wide.

example role definition which grants access to pods in default namespace:

```apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```
2. RoleBinding and ClusterRoleBinding
A role binding grants the permissions defined in a role to a subject (users, groups, or service accounts)

Example ClusterRoleBinding that grants the "pod-reader" Role to the user "jane" within the "default" namespace. This allows "jane" to read pods in the "default" namespace.
```
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

### APIs
### Multitenancy and cluster hardening
### Secret
Official documentation: <https://kubernetes.io/docs/concepts/configuration/secret/>

A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a Pod specification or in an image. Users can create Secrets and the system also creates some Secrets.

Note: Kubernetes Secrets are, by default, stored as unencrypted base64-encoded strings. Those can be directly accessed on etcd or anyone with a relevant API access

There are following types of secrets available in kubernetes:

* Opaque	arbitrary user-defined data
* kubernetes.io/service-account-token	service account token
* kubernetes.io/dockercfg	serialized ~/.dockercfg file
* kubernetes.io/dockerconfigjson	serialized ~/.docker/config.json file
* kubernetes.io/basic-auth	credentials for basic authentication
* kubernetes.io/ssh-auth	credentials for SSH authentication
* kubernetes.io/tls	data for a TLS client or server
* bootstrap.kubernetes.io/token	bootstrap token data

### ConfigMap
Official documentation: <https://kubernetes.io/docs/concepts/configuration/configmap/>

A ConfigMap is an API object used to store non-confidential data in key-value pairs. Pods can consume ConfigMaps as environment variables, command-line arguments, or as configuration files in a volume.

There are four different ways that you can use a ConfigMap to configure a container inside a Pod:

* Inside a container command and args
* Environment variables for a container
* Add a file in read-only volume, for the application to read
* Write code to run inside the Pod that uses the Kubernetes API to read a ConfigMap

### Admission controllers
Using admission controllers: <https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/>
admission controllers is a piece of code which intercepts API requests (after those are authenticated and authorized)
There is a whole list of controllers, of which two are special - MutatingAdmissionWebhook and ValidatingAdmissionWebhook.
With admission controllers one can change the bahaviour of created object, e.g. AlwaysPullImages or DefaultIngressClass.
For example cert-manager uses MutatingAdmissionWebhook and ValidatingAdmissionWebhook for cainjector


### Ingress solutions

Kubernetes as a project supports and maintains AWS, GCE, and nginx ingress controllers but there are many controllers available provided by third parties, e.g. ha-proxy, istio etc.

each cluster must define at least one ingress controller to be usable. There is a possiblity to run more than one ingress controllers and choose which one to use by the app with defining ingress.class. Example could be to run multiple instances of nginx ingress controllers with a different configuration or nginx controller with ha-proxy.



### CRDs
Official documentation: <https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/>

Custom resources are extensions of the Kubernetes API. CRD allows to define kubernetes resources with top-level support from kubectl, kubectl get myobject objectname

Kubernetes provides two ways to add custom resources to your cluster:

* CRDs are simple and can be created without any programming.
* API Aggregation requires programming, but allows more control over API behaviors like how data is stored and conversion between API versions.

The CustomResourceDefinition API resource allows you to define custom resources. Defining a CRD object creates a new custom resource with a name and schema that you specify. The Kubernetes API serves and handles the storage of your custom resource. The name of a CRD object must be a valid DNS subdomain name.

This frees you from writing your own API server to handle the custom resource, but the generic nature of the implementation means you have less flexibility than with API server aggregation.

Refer to the custom controller example <https://github.com/kubernetes/sample-controller> for an example of how to register a new custom resource, work with instances of your new resource type, and use a controller to handle events.

Extend k8s API with CRDs: <https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/>





## Programming
### Optimisations
### Memory issues 
### Cache algorithms
### Code performance
### Data structures
#### Double linked lists
#### Arrays
### Code availability and reliability
### Pointers
### Systems level architecture of distributed or similar systems
### High performance computing
### Time/space complexity
### Test Driven development 
### Edge cases
### CI/CD
### Web requests
### Multithreading

## Systems
### Designing a monitoring system
### OOM and resource limits
### Aggregations
### Storage
### Signals
### Boot processes
### Command lines
### Upgrades
### Patches
### Server builds

## Other
### GitOps
### Different types of testing
### IaC tools
### SQL and NoSQL databases
### Networking fundamentals

