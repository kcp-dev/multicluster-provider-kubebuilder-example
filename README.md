# multicluster-provider-kubebuilder-example

A full kcp multicluster-provider example using kubebuilder.

This example demonstrates how to lift&shift a kubebuilder project to a multicluster-provider project for kcp.

There should be 2 commits in this repository at all times:

1. The initial commit with the kubebuilder project.

Skeleton created by running [kubebuilder](https://book.kubebuilder.io/quick-start.html) with the following command:

```sh
$ kubebuilder init --domain contrib.kcp.io --repo github.com/kcp-dev/multicluster-provider-kubebuilder-example
$ kubebuilder create api --group apis --version v1alpha1 --kind Application
```

2. The commit with the kubebuilder project lifted&shifted to a multicluster-provider project to work with kcp.

Please refer to main repositories for more information:

- [multicluster-provider for kcp](https://github.com/kcp-dev/multicluster-provider)
- [multicluster-runtime](https://github.com/multicluster-runtime/multicluster-runtime)
- [kcp](https://github.com/kcp-dev/kcp)


## Customizations & Run

### Prerequisites

- [kcp](https://github.com/kcp-dev/kcp) installed and running
- [kcp krew plugins](https://docs.kcp.io/kcp/v0.26/setup/kubectl-plugin/) installed

1. Make file targets to install kcp specific tooling and generate kcp specific manifests:

```sh
# install tooling
make kcp-tools
# generate kcp manifests
make kcp-generate
```

After than content of `cmd/main.go` was updated with multicluster extension.

It can be tested by applying the necessary manifests from the respective folder while connected to the `root` workspace of a kcp instance:

export KUBECONFIG=admin.kubeconfig

```sh
kubectl ws create provider --enter
kubectl apply -f ./config/kcp/apiresourceschema-applications.apis.contrib.kcp.io.yaml
kubectl apply -f ./config/kcp/apiexport-apis.contrib.kcp.io.yaml

# Consumer 1
kubectl ws use :root
kubectl ws create examples-kubebuilder-multicluster-1 --enter
kubectl kcp bind apiexport root:provider:apis.contrib.kcp.io 
kubectl apply -f ./config/samples/apis_v1alpha1_application.yaml

# Consumer 2
kubectl ws use :root
kubectl ws create examples-kubebuilder-multicluster-2 --enter
kubectl kcp bind apiexport root:provider:apis.contrib.kcp.io 
kubectl apply -f ./config/samples/apis_v1alpha1_application.yaml
```

Then, start the example controller by passing the virtual workspace URL to it:

```sh
kubectl ws use :root:provider
make run
```

Observe the controller reconciling the ` application-sample` Aplication in the workspaces:

```sh
2025-03-11T13:04:52+02:00       INFO    Reconciling Application {"controller": "kcp-applications-controller", "controllerGroup": "apis.contrib.kcp.io", "controllerKind": "Application", "reconcileID": "babfc696-50cc-4851-ab35-d1d956a6c120", "cluster": "1058d5hgzdd3ask6"}
```