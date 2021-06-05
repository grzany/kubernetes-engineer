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
### Pods
### Nodes
See [here](#components)
### Requests
### Metrics and logging
### Troubleshooting
### Accessing applications hosted in Kubernetes
### Pod controllers
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
### CPU and memory limits
### Multitenancy and cluster hardening
### Secrets/ConfigMaps
### Admission controllers
Using admission controllers: <https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/>
admission controllers is a piece of code which intercepts API requests (after those are authenticated and authorized)
There is a whole list of controllers, of which two are special - MutatingAdmissionWebhook and ValidatingAdmissionWebhook.
With admission controllers one can change the bahaviour of created object, e.g. AlwaysPullImages or DefaultIngressClass.
For example cert-manager uses MutatingAdmissionWebhook and ValidatingAdmissionWebhook for cainjector


### Ingress solutions
### CRDs

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

