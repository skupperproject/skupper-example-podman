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
* [Step 1: Install the Skupper command-line tool](#step-1-install-the-skupper-command-line-tool)
* [Step 2: Set up your kubeconfigs](#step-2-set-up-your-kubeconfigs)
* [Step 3: Set up your Kubernetes sites](#step-3-set-up-your-kubernetes-sites)
* [Step 4: Set up your Podman site](#step-4-set-up-your-podman-site)
* [Step 5: Link your sites](#step-5-link-your-sites)
* [Step 6: Deploy the application workloads](#step-6-deploy-the-application-workloads)
* [Step 7: Expose the backend service](#step-7-expose-the-backend-service)
* [Step 8: Access the application](#step-8-access-the-application)
* [Cleaning up](#cleaning-up)
* [Summary](#summary)
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

[install-script]: https://github.com/skupperproject/skupper-website/blob/main/input/install.sh
[install-docs]: https://skupper.io/install/

## Step 2: Set up your kubeconfigs

Skupper is designed for use with multiple namespaces, usually on
different clusters.  The `skupper` and `kubectl` commands use your
[kubeconfig][kubeconfig] and current context to select the
namespace where they operate.

[kubeconfig]: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/

Your kubeconfig is stored in a file in your home directory.  The
`skupper` and `kubectl` commands use the `KUBECONFIG` environment
variable to locate it.

A single kubeconfig supports only one active context per user.
Since you will be using multiple contexts at once in this
exercise, you need to create distinct kubeconfigs.

For each namespace, open a new terminal window.  In each
terminal, set the `KUBECONFIG` environment variable to a different
path and log in to your cluster.

_**Kubernetes:**_

~~~ shell
export KUBECONFIG=~/.kube/config-hello-world
# Enter your provider-specific login command

~~~

The login procedure varies by provider.  See the documentation for
your chosen providers:

* [Minikube](https://skupper.io/start/minikube.html#cluster-access)
* [Amazon Elastic Kubernetes Service (EKS)](https://skupper.io/start/eks.html#cluster-access)
* [Azure Kubernetes Service (AKS)](https://skupper.io/start/aks.html#cluster-access)
* [Google Kubernetes Engine (GKE)](https://skupper.io/start/gke.html#cluster-access)
* [IBM Kubernetes Service](https://skupper.io/start/ibmks.html#cluster-access)
* [OpenShift](https://skupper.io/start/openshift.html#cluster-access)

## Step 3: Set up your Kubernetes sites

For each site, create the namespace you wish to use (or use an
existing namespace).  Set the namespace on your current context.
Use `skupper init` to install Skupper in the current namespace.

**Note:** If you are using Minikube, you need to [start minikube
tunnel][minikube-tunnel] before you install Skupper.

[minikube-tunnel]: https://skupper.io/start/minikube.html#running-minikube-tunnel

_**Kubernetes:**_

~~~ shell
kubectl create namespace hello-world
kubectl config set-context --current --namespace hello-world
skupper init --enable-console --enable-flow-collector
~~~

_Sample output:_

~~~ console
$ skupper init
Waiting for LoadBalancer IP or hostname...
Waiting for status...
Skupper is now installed in namespace '<namespace>'.  Use 'skupper status' to get more information.
~~~

## Step 4: Set up your Podman site

Open a new terminal window and set the `SKUPPER_PLATFORM`
environment variable to `podman`.

Make sure the Podman API service is available.  On most systems
you can use:

~~~
systemctl --user enable --now podman.socket
~~~

You can also use:

~~~
podman system service --time=0 unix://$XDG_RUNTIME_DIR/podman/podman.sock &
~~~

Then use `skupper init` to install Skupper in your Podman environment.

_**Podman:**_

~~~ shell
export SKUPPER_PLATFORM=podman
systemctl --user enable --now podman.socket
skupper init --ingress none
~~~

_Sample output:_

~~~ console
$ skupper init --ingress none
Skupper is now installed for user 'jross'.  Use 'skupper status' to get more information.
~~~

## Step 5: Link your sites

Creating a link requires use of two `skupper` commands in
conjunction, `skupper token create` and `skupper link create`.

The `skupper token create` command generates a secret token that
signifies permission to create a link.  The token also carries the
link details.  Then, in a remote site, The `skupper link
create` command uses the token to create a link to the site
that generated it.

**Note:** The link token is truly a *secret*.  Anyone who has the
token can link to your site.  Make sure that only those you trust
have access to it.

First, use `skupper token create` in one site to generate the
token.  Then, use `skupper link create` in another to link the two
sites.

_**Kubernetes:**_

~~~ shell
skupper token create ~/secret.token
~~~

_Sample output:_

~~~ console
$ skupper token create ~/secret.token
Token written to ~/secret.token
~~~

_**Podman:**_

~~~ shell
skupper link create ~/secret.token
~~~

_Sample output:_

~~~ console
$ skupper link create ~/secret.token
Site configured to link to https://10.105.193.154:8081/ed9c37f6-d78a-11ec-a8c7-04421a4c5042 (name=link1)
Check the status of the link using 'skupper link status'.
~~~

If your terminal sessions are on different machines, you may need
to use `scp` or a similar tool to transfer the token securely.  By
default, tokens expire after a single use or 15 minutes after
creation.

## Step 6: Deploy the application workloads

This example runs the frontend on Kubernetes and the backend as
a local Podman container.

Use `kubectl create deployment` to deploy the frontend service
in `hello-world`.

Use `docker run` or `podman run` to run the backend service on
your local machine.

_**Kubernetes:**_

~~~ shell
kubectl create deployment frontend --image quay.io/skupper/hello-world-frontend
~~~

_**Podman:**_

~~~ shell
podman run --name backend-target --network skupper --detach --rm -p 8080:8080 quay.io/skupper/hello-world-backend
~~~

## Step 7: Expose the backend service

XXX

Use `skupper service create` to define a Skupper service called
`backend`.  Then use `skupper gateway bind` to attach your
running backend process as a target for the service.

XXX

_**Kubernetes:**_

~~~ shell
skupper service create backend 8080
~~~

_**Podman:**_

~~~ shell
skupper service create backend 8080
skupper service bind backend host backend-target --target-port 8080
~~~

## Step 8: Access the application

In order to use and test the application, we need external access
to the frontend.

Use `kubectl expose` with `--type LoadBalancer` to open network
access to the frontend service.

Once the frontend is exposed, use `kubectl get service/frontend`
to look up the external IP of the frontend service.  If the
external IP is `<pending>`, try again after a moment.

Once you have the external IP, use `curl` or a similar tool to
request the `/api/health` endpoint at that address.

**Note:** The `<external-ip>` field in the following commands is a
placeholder.  The actual value is an IP address.

_**Kubernetes:**_

~~~ shell
kubectl expose deployment/frontend --port 8080 --type LoadBalancer
kubectl get service/frontend
curl http://<external-ip>:8080/api/health
~~~

_Sample output:_

~~~ console
$ kubectl expose deployment/frontend --port 8080 --type LoadBalancer
service/frontend exposed

$ kubectl get service/frontend
NAME       TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)          AGE
frontend   LoadBalancer   10.103.232.28   <external-ip>   8080:30407/TCP   15s

$ curl http://<external-ip>:8080/api/health
OK
~~~

If everything is in order, you can now access the web interface by
navigating to `http://<external-ip>:8080/` in your browser.

## Cleaning up

To remove Skupper and the other resources from this exercise, use
the following commands.

_**Kubernetes:**_

~~~ shell
skupper delete
kubectl delete service/frontend
kubectl delete deployment/frontend
~~~

_**Podman:**_

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
