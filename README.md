# Skupper Hello World using Podman

[![main](https://github.com/ssorj/skupper-example-podman/actions/workflows/main.yaml/badge.svg)](https://github.com/ssorj/skupper-example-podman/actions/workflows/main.yaml)

#### Connect services running as Podman containers

This example is part of a [suite of examples][examples] showing the
different ways you can use [Skupper][website] to connect services
across cloud providers, data centers, and edge sites.

[website]: https://skupper.io/
[examples]: https://skupper.io/examples/index.html

#### Contents

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Step 1: Install the Skupper command-line tool](#step-1-install-the-skupper-command-line-tool)
* [Step 2: Access your Kubernetes cluster](#step-2-access-your-kubernetes-cluster)
* [Step 3: Configure separate console sessions](#step-3-configure-separate-console-sessions)
* [Step 4: Set up your Kubernetes namespace](#step-4-set-up-your-kubernetes-namespace)
* [Step 5: Set up your Podman environment](#step-5-set-up-your-podman-environment)
* [Step 6: Install Skupper in your sites](#step-6-install-skupper-in-your-sites)
* [Step 7: Link your sites](#step-7-link-your-sites)
* [Step 8: Deploy the frontend and backend services](#step-8-deploy-the-frontend-and-backend-services)
* [Step 9: Expose the backend service](#step-9-expose-the-backend-service)
* [Step 10: Expose the frontend service](#step-10-expose-the-frontend-service)
* [Cleaning up](#cleaning-up)
* [About this example](#about-this-example)

## Overview

This example is a basic multi-service HTTP application deployed
across a Kubernetes cluster and a bare-metal host or VM running
Podman containers.

It contains two services:

* A backend service that exposes an `/api/hello` endpoint.  It
  returns greetings of the form `Hi, <your-name>.  I am <my-name>
  (<container>)`.

* A frontend service that sends greetings to the backend and
  fetches new greetings in response.

With Skupper, you can run the backend as a container on your local
machine and the frontend in Kubernetes and maintain connectivity
between the two services without exposing the backend to the public
internet.

<!-- <img src="images/entities.svg" width="640"/> -->

## Prerequisites

* A working installation of Podman ([installation
  guide][install-podman])

* The `kubectl` command-line tool, version 1.15 or later
  ([installation guide][install-kubectl])

* Access to a Kubernetes cluster, from [any provider you
  choose][kube-providers]

[install-podman]: https://podman.io/getting-started/installation
[install-kubectl]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[kube-providers]: https://skupper.io/start/kubernetes.html

## Step 1: Install the Skupper command-line tool

The `skupper` command-line tool is the entrypoint for installing
and configuring Skupper.  You need to install the `skupper`
command only once for each development environment.

On Linux or Mac, you can use the install script (inspect it
[here][install-script]) to download and extract the command:

~~~ shell
curl https://skupper.io/install.sh | sh
~~~

The script installs the command under your home directory.  It
prompts you to add the command to your path if necessary.

For Windows and other installation options, see [Installing
Skupper][install-docs].

[install-script]: https://github.com/skupperproject/skupper-website/blob/main/docs/install.sh
[install-docs]: https://skupper.io/install/index.html

## Step 2: Access your Kubernetes cluster

The procedure for accessing a Kubernetes cluster varies by
provider. [Find the instructions for your chosen
provider][kube-providers] and use them to authenticate and
configure access.

## Step 3: Configure separate console sessions

_**Console for Kubernetes:**_

~~~ shell
export KUBECONFIG=@kubeconfig@
~~~

_**Console for Podman:**_

~~~ shell
export SKUPPER_PLATFORM=podman
~~~

## Step 4: Set up your Kubernetes namespace

Use `kubectl create namespace` to create the namespace you wish
to use (or use an existing namespace).  Use `kubectl config
set-context` to set the current namespace for your session.

_**Console for Kubernetes:**_

~~~ shell
kubectl create namespace hello-world
kubectl config set-context --current --namespace hello-world
~~~

_Sample output:_

~~~ console
$ kubectl create namespace hello-world
namespace/hello-world created

$ kubectl config set-context --current --namespace hello-world
Context "minikube" modified.
~~~

## Step 5: Set up your Podman environment

_**Console for Podman:**_

~~~ shell
podman system service --time=0 unix://$XDG_RUNTIME_DIR/podman/podman.sock &
~~~

## Step 6: Install Skupper in your sites

_**Console for Kubernetes:**_

~~~ shell
skupper init --enable-console --enable-flow-collector
~~~

_**Console for Podman:**_

~~~ shell
skupper init --ingress none
~~~

## Step 7: Link your sites

_**Console for Kubernetes:**_

~~~ shell
skupper token create ~/secret.token
~~~

_**Console for Podman:**_

~~~ shell
skupper link create ~/secret.token
~~~

## Step 8: Deploy the frontend and backend services

This example runs the frontend on Kubernetes and the backend as
a local Podman container.

Use `kubectl create deployment` to deploy the frontend service
in `hello-world`.

Use `docker run` or `podman run` to run the backend service on
your local machine.

_**Console for Kubernetes:**_

~~~ shell
kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
~~~

_Sample output:_

~~~ console
$ kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
deployment.apps/frontend created
~~~

_**Console for Podman:**_

~~~ shell
podman run --name backend-target --network skupper --detach --rm -p 8080:8080 quay.io/skupper/hello-world-backend
~~~

_Sample output:_

~~~ console
$ podman run --name backend-target --network skupper --detach --rm -p 8080:8080 quay.io/skupper/hello-world-backend
262dde0287af2c76c3088d9ff4f865f02732a762b0afd91e03ec9e3fe6b03f88
~~~

## Step 9: Expose the backend service

Use `skupper service create` to define a Skupper service called
`backend`.  Then use `skupper gateway bind` to attach your
running backend process as a target for the service.

_**Console for Kubernetes:**_

~~~ shell
skupper service create backend 8080
~~~

_**Console for Podman:**_

~~~ shell
skupper service create backend 8080
skupper service bind backend host backend-target --target-port 8080
~~~

## Step 10: Expose the frontend service

We have established connectivity between the Kubernetes
namespace and the your local machine, and we've made the backend
in `hello-world` available to the frontend running as container.
Before we can test the application, we need external access to
the frontend.

Use `kubectl expose` with `--type LoadBalancer` to open network
access to the frontend service.

_**Console for Kubernetes:**_

~~~ shell
kubectl expose deployment/frontend --port 8080 --type LoadBalancer
~~~

_Sample output:_

~~~ console
$ kubectl expose deployment/frontend --port 8080 --type LoadBalancer
service/frontend exposed
~~~

## Cleaning up

To remove Skupper and the other resources from this exercise, use
the following commands.

_**Console for Kubernetes:**_

~~~ shell
skupper delete
kubectl delete service/frontend
kubectl delete deployment/frontend
~~~

_**Console for Podman:**_

~~~ shell
skupper delete
~~~

## Next steps

Check out the other [examples][examples] on the Skupper website.

## About this example

This example was produced using [Skewer][skewer], a library for
documenting and testing Skupper examples.

[skewer]: https://github.com/skupperproject/skewer

Skewer provides utility functions for generating the README and
running the example steps.  Use the `./plano` command in the project
root to see what is available.

To quickly stand up the example using Minikube, try the `./plano demo`
command.
