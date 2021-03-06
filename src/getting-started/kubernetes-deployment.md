# Kubernetes deployment

The [`deployment`](https://github.com/chimera-kube/chimera-admission/tree/main/deployment)
directory inside of the [chimera-admission](https://github.com/chimera-kube/chimera-admission)
repository shows how to deploy the admission controller on top of a Kubernetes
cluster.

chimera-admission can be deployed by doing a simple:

```shell
$ kubectl apply -f https://raw.githubusercontent.com/chimera-kube/chimera-admission/main/deployment/chimera-admission.yaml
```
The chimera-admission controller will automatically register itself against the
Kubernetes API server

> **Note well:** `chimera-admission` is still in alpha phase. It's not meant to
> be used in production.
>
> We also plan to create a Kubernetes controller to simplify the management
> of Chimera Policies.

## Components

The YAML file takes care of deploying all the required Kubernetes resources.
The next section covers in detail all the objects that will be created.

### Namespace

A `Namespace` called `chimera` is created. All the other resources are
placed inside of it.

### ServiceAccount

A `ServiceAccount` named `chimera` is created. This is used by the
`chimera-admission` Pod to authenticate against the local Kubernetes API.

### RBAC policies

A `ClusterRole` and a `ClusterRoleBinding` objects are created.

These are used to grant the right set of privileges to the previously created
service account.

The service account is granted the right to do any kind of operation
against `admissionregistration.k8s.io/validatingwebhookconfigurations`
resources.

This is required to allow `chimera-admission` to register itself against the
local Kubernetes API server and, eventually, remove old instances of itself.

### Deployment

A `Deployment` named `chimera-admission` is created.

The deployment uses the [container image](https://github.com/orgs/chimera-kube/packages/container/package/chimera-admission)
we push automatically to our GitHub Container Registry.

The admission controller uses the [pod-toleration](https://github.com/chimera-kube/pod-toleration-policy)
policy to validate incoming Pod requests.

The controller will download the Wasm module providing the policy from
[here](https://github.com/orgs/chimera-kube/packages/container/package/policies%2Fpod-toleration).

chimera-admission will automatically generate a certificate authority and,
through it, will sign a certificate. This certificate is used by the
controller to secure the communication with the Kubernetes API server.

> **Note well:** right now chimera-admission doesn't support automatic certificate
> rotation. It's however possible to provide a custom TLS certificate and an
> already existing CA bundle to be used.

### Service

A `Service` named `chimera-admission` is created. This is used to expose the
`chimera-admission` Deployment inside of the cluster.

This is how the Kubernetes API reaches `chimera-admission` to validate
incoming requests.
