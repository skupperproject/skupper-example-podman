<!-- NOTE: This file is generated from skewer.yaml.  Do not edit it directly. -->

# Skupper Hello World using Podman

#### Connect services running as Podman containers

This example is part of a [suite of examples][examples] showing the
different ways you can use [Skupper][website] to connect services
across cloud providers, data centers, and edge sites.

[website]: https://skupper.io/
[examples]: https://skupper.io/examples/index.html

#### Contents

* [Overview](#overview)
* [Prerequisites](#prerequisites)
* [Step 1: Set up your Kubernetes cluster](#step-1-set-up-your-kubernetes-cluster)
* [Step 2: Set up your Podman environment](#step-2-set-up-your-podman-environment)
* [Step 3: Install Skupper on your clusters](#step-3-install-skupper-on-your-clusters)
* [Step 4: Deploy the frontend and backend](#step-4-deploy-the-frontend-and-backend)
* [Step 5: Create your sites](#step-5-create-your-sites)
* [Step 6: Sleep!](#step-6-sleep)
* [Next steps](#next-steps)
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

## Step 1: Set up your Kubernetes cluster

Open a new terminal window and log in to your cluster.  Then
create the namespace you wish to use and set the namespace on your
current context.

**Note:** The login procedure varies by provider.  See the
documentation for your chosen providers:

* [Minikube](https://skupper.io/start/minikube.html#cluster-access)
* [Amazon Elastic Kubernetes Service (EKS)](https://skupper.io/start/eks.html#cluster-access)
* [Azure Kubernetes Service (AKS)](https://skupper.io/start/aks.html#cluster-access)
* [Google Kubernetes Engine (GKE)](https://skupper.io/start/gke.html#cluster-access)
* [IBM Kubernetes Service](https://skupper.io/start/ibmks.html#cluster-access)
* [OpenShift](https://skupper.io/start/openshift.html#cluster-access)

_**Kubernetes:**_

~~~ shell
# Enter your provider-specific login command
kubectl create namespace hello-world
kubectl config set-context --current --namespace hello-world
~~~

## Step 2: Set up your Podman environment

Open a new terminal window and set the `SKUPPER_PLATFORM`
environment variable to `podman`.  This sets the Skupper platform
to Podman for this terminal session.

Use `podman network create` to create the Podman network that
Skupper will use.

Use `systemctl` to enable the Podman API service.

_**Podman:**_

~~~ shell
export SKUPPER_PLATFORM=podman
podman network create skupper
systemctl --user enable --now podman.socket
~~~

If the `systemctl` command doesn't work, you can try the `podman
system service` command instead:

~~~
podman system service --time 0 unix://$XDG_RUNTIME_DIR/podman/podman.sock &
~~~

## Step 3: Install Skupper on your clusters

Using Skupper on Kubernetes requires the installation of the
Skupper custom resource definitions (CRDs) and the Skupper
controller.

For each cluster, use `kubectl apply` with the Skupper
installation YAML to install the CRDs and controller.

_**Kubernetes:**_

~~~ shell
kubectl apply -f https://skupper.io/v2/install.yaml
~~~

## Step 4: Deploy the frontend and backend

This example runs the frontend in Kubernetes and the backend as
a local Podman container.

In Kubernetes, use `kubectl create deployment` to deploy the
frontend service in namespace `hello-world`.

In Podman, use `podman run` to start the backend service on your
local machine.

_**Kubernetes:**_

~~~ shell
kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
~~~

_**Podman:**_

~~~ shell
podman run --name backend --detach --rm -p 8080:8080 quay.io/skupper/hello-world-backend
~~~

## Step 5: Create your sites

A Skupper _site_ is a location where components of your
application are running.  Sites are linked together to form a
Skupper network for your application.

In Kubernetes, use `skupper site create` with a site name of
your choice.  This creates the site resource and deploys the
Skupper router to the namespace.

XXX In Podman, use `skupper init` with the option `--ingress none`
and use `skupper status` to see the result. XXX

**Note:** If you are using Minikube, you need to [start minikube
tunnel][minikube-tunnel] before you run `skupper init`.

<!-- XXX Explain enabling link acesss on one of the sites -->

[minikube-tunnel]: https://skupper.io/start/minikube.html#running-minikube-tunnel

_**Kubernetes:**_

~~~ shell
skupper site create hello-world --enable-link-access
kubectl wait --for condition=Ready site/hello-world  # Required with preview 1 - to be removed!
~~~

_Sample output:_

~~~ console
$ skupper site create hello-world --enable-link-access
Waiting for status...
Site "hello-world" is configured. Check the status to see when it is ready

$ kubectl wait --for condition=Ready site/hello-world  # Required with preview 1 - to be removed!
site.skupper.io/hello-world condition met
~~~

_**Podman:**_

~~~ shell
echo XXX
~~~

_Sample output:_

~~~ console
$ echo XXX
XXX
~~~

You can use `skupper site status` at any time to check the status
of your site.

## Step 6: Sleep!

_**Kubernetes:**_

~~~ shell
sleep 10
kubectl get pods
podman ps
sleep 86400
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
