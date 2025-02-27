# sample-cli-plugin

This repository implements a single kubectl plugin for switching the namespace
that the current KUBECONFIG context points to. In order to remain as indestructive
as possible, no existing contexts are modified.

**Note:** go-get or vendor this package as `k8s.io/sample-cli-plugin`.

This particular example demonstrates how to perform basic operations such as:

* How to create a new custom command that follows kubectl patterns
* How to obtain a user's KUBECONFIG settings and modify them
* How to make general use of the provided "cli-runtime" set of helpers for kubectl and third-party plugins

It makes use of the genericclioptions in [k8s.io/cli-runtime](https://github.com/kubernetes/cli-runtime)
to generate a set of configuration flags which are in turn used to generate a raw representation of
the user's KUBECONFIG, as well as to obtain configuration which can be used with RESTClients when sending
requests to a kubernetes api server.

## Details

The sample cli plugin uses the [client-go library](https://github.com/kubernetes/client-go/tree/master/tools/clientcmd) to patch an existing KUBECONFIG file in a user's environment in order to update context information to point the client to a new or existing namespace.

In order to be as non-destructive as possible, no existing contexts are modified in any way. Rather, the current context is examined, and matched against existing contexts to find a context containing the same "AuthInfo" and "Cluster" information, but with the newly desired namespace requested by the user.

## Purpose

This is an example of how to build a kubectl plugin using the same set of tools and helpers available to kubectl.

## Installation

### Manual

```sh
# assumes you have a working KUBECONFIG
$ GO111MODULE="on" go build cmd/kubectl-ns.go
# place the built binary somewhere in your PATH
$ cp ./kubectl-ns /usr/local/bin
```

##### Cleanup

You can "uninstall" this plugin from kubectl by simply removing it from your PATH:

    $ rm /usr/local/bin/kubectl-ns

### Krew

Krew is a kubectl plugin manager which simplifies the plugin installation.
Visit [krew.dev](https://krew.dev/#installation) to set up krew.

```sh
# build the plugin archive to be used by krew
$ hack/create-plugin-archive.sh
# generate a ready-to-use krew manifest from the template
$ hack/generate-krew-manifest.sh
# install the plugin via krew
$ kubectl krew install --manifest out/sample-plugin.yaml --archive out/archive.tar.gz
```

##### Cleanup

You can uninstall this plugin via krew:

    $ kubectl krew remove sample-plugin

## Running

```sh
# After installation, you can use this plugin as a regular kubectl command:
# update your configuration to point to "new-namespace"
$ kubectl ns new-namespace
# any kubectl commands you perform from now on will use "new-namespace"
$ kubectl get pod
NAME                READY     STATUS    RESTARTS   AGE
new-namespace-pod   1/1       Running   0          1h

# list all of the namespace in use by contexts in your KUBECONFIG
$ kubectl ns --list

# show the namespace that the currently set context in your KUBECONFIG points to
$ kubectl ns
```

## Use Cases

This plugin can be used as a developer tool, in order to quickly view or change the current namespace
that kubectl points to.

It can also be used as a means of showcasing usage of the cli-runtime set of utilities to aid in
third-party plugin development.

## Compatibility

HEAD of this repository will match HEAD of k8s.io/apimachinery and
k8s.io/client-go.

## Where does it come from?

`sample-cli-plugin` is synced from
https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/sample-cli-plugin.
Code changes are made in that location, merged into k8s.io/kubernetes and
later synced here.
