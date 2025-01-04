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
* [Step 3: Deploy the frontend and backend](#step-3-deploy-the-frontend-and-backend)
* [Step 4: Install the Skupper command-line tool](#step-4-install-the-skupper-command-line-tool)
* [Step 5: Install Skupper on your Kubernetes cluster](#step-5-install-skupper-on-your-kubernetes-cluster)
* [Step 6: Install Skupper in your Podman environment](#step-6-install-skupper-in-your-podman-environment)
* [Step 7: Create your sites](#step-7-create-your-sites)
* [Step 8: Link your sites](#step-8-link-your-sites)
* [Step 9: Expose the backend service](#step-9-expose-the-backend-service)
* [Step 10: Access the frontend service](#step-10-access-the-frontend-service)
* [Cleaning up](#cleaning-up)
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
kubectl create namespace west
kubectl config set-context --current --namespace west
~~~

## Step 2: Set up your Podman environment

Open a new terminal window and set the `SKUPPER_PLATFORM`
environment variable to `podman`.  This sets the Skupper platform
to Podman for this terminal session.

Use `systemctl` to enable the Podman API service.

_**Podman:**_

~~~ shell
export SKUPPER_PLATFORM=podman
~~~

## Step 3: Deploy the frontend and backend

This example runs the frontend in Kubernetes and the backend as
a local Podman container.

In Kubernetes, use `kubectl create deployment` to deploy the
frontend service in namespace `west`.

In Podman, use `podman run` to start the backend service on your
local machine.

_**Kubernetes:**_

~~~ shell
kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
~~~

_**Podman:**_

~~~ shell
podman run --name backend --detach --rm -p 9090:8080 quay.io/skupper/hello-world-backend
~~~

## Step 4: Install the Skupper command-line tool

This example uses the Skupper command-line tool to create Skupper
resources.  You need to install the `skupper` command only once
for each development environment.

On Linux or Mac, you can use the install script (inspect it
[here][install-script]) to download and extract the command:

~~~ shell
curl https://skupper.io/install.sh | sh -s -- --version 2.0.0-preview-2
~~~

The script installs the command under your home directory.  It
prompts you to add the command to your path if necessary.

For Windows and other installation options, see [Installing
Skupper][install-docs].

[install-script]: https://github.com/skupperproject/skupper-website/blob/main/input/install.sh
[install-docs]: https://skupper.io/install/

## Step 5: Install Skupper on your Kubernetes cluster

Using Skupper on Kubernetes requires the installation of the
Skupper custom resource definitions (CRDs) and the Skupper
controller.

Use `kubectl apply` with the Skupper installation YAML to install
the CRDs and controller.

_**Kubernetes:**_

~~~ shell
kubectl apply -f https://github.com/skupperproject/skupper/releases/download/2.0.0-preview-2/skupper-setup-cluster-scope.yaml
~~~

## Step 6: Install Skupper in your Podman environment

_**Podman:**_

~~~ shell
# Current:
systemctl --user enable --now podman.socket
# Want:
# skupper system install
# skupper system start
~~~

## Step 7: Create your sites

A Skupper _site_ is a location where your application workloads
are running.  Sites are linked together to form a network for your
application.

For each namespace, use `skupper site create` with a site name of
your choice.  This creates the site resource and deploys the
Skupper router to the namespace.

**Note:** If you are using Minikube, you need to [start minikube
tunnel][minikube-tunnel] before you run `skupper site create`.

<!-- XXX Explain enabling link acesss on one of the sites -->

[minikube-tunnel]: https://skupper.io/start/minikube.html#running-minikube-tunnel

_**Kubernetes:**_

~~~ shell
skupper site create west --enable-link-access --timeout 2m
~~~

_Sample output:_

~~~ console
$ skupper site create west --enable-link-access --timeout 2m
Waiting for status...
Site "west" is configured. Check the status to see when it is ready
~~~

_**Podman:**_

~~~ shell
skupper site create east
# Current:
skupper system setup
# Want:
# skupper system reload
~~~

## Step 8: Link your sites

A Skupper _link_ is a channel for communication between two sites.
Links serve as a transport for application connections and
requests.

_**Kubernetes:**_

~~~ shell
# Current:
skupper link generate > link.yaml
# Want:
# skupper token issue ~/secret.token
~~~

_**Podman:**_

~~~ shell
# Current:
mv link.yaml /home/jross/.local/share/skupper/namespaces/default/input/resources/link.yaml
# Want:
# skupper token redeem ~/secret.token
skupper system reload
~~~

## Step 9: Expose the backend service

We now have our sites linked to form a Skupper network, but no
services are exposed on it.

Skupper uses _listeners_ and _connectors_ to expose services
across sites inside a Skupper network.  A listener is a local
endpoint for client connections, configured with a routing key.  A
connector exists in a remote site and binds a routing key to a
particular set of servers.  Skupper routers forward client
connections from local listeners to remote connectors with
matching routing keys.

In Kubernetes, use the `skupper listener create` command to create a
listener for the backend.  In Podman, use the `skupper connector
create` command to create a matching connector.

_**Kubernetes:**_

~~~ shell
skupper listener create backend 8080
~~~

_**Podman:**_

~~~ shell
# Current:
skupper connector create backend 9090 --host localhost
# Want:
# skupper connector create backend 9090
skupper system reload
~~~

The commands shown above use the name argument, `backend`, to
also set the default routing key.  You can use the
`--routing-key` option to specify another value.

## Step 10: Access the frontend service

In order to use and test the application, we need external access
to the frontend.

Use `kubectl port-forward` to make the frontend available at
`localhost:8080`.

_**Kubernetes:**_

~~~ shell
kubectl port-forward deployment/frontend 8080:8080
~~~

You can now access the web interface by navigating to
[http://localhost:8080](http://localhost:8080) in your browser.

## Cleaning up

To remove Skupper and the other resources from this exercise, use
the following commands.

_**Kubernetes:**_

~~~ shell
skupper site delete --all
~~~

_**Podman:**_

~~~ shell
skupper site delete --all
podman rm -f backend
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
