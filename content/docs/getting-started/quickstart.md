---
title: "Quickstart"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 1
---

# Quickstart

In this guide, we’ll walk you through how to install Marblerun into your Kubernetes cluster. Then we’ll deploy a sample confidential application to demonstrate the capabilities of Marblerun.

Installing Marblerun is easy. First, you will install the CLI (command-line interface) onto your local machine. Using this CLI, you’ll then install the control plane onto your Kubernetes cluster.
Finally, you’ll "mesh" one or more of your own services and setup a Manifest defining your confidential service mesh.

## Step 0: Setup

Before we can do anything, we need to ensure you have access to a Kubernetes cluster and a functioning kubectl command on your local machine. Probably the easiest way to get started is to run Kubernetes on your local machine using [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/). Another easy way is to use [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal), which offers SGX-enabled nodes.

You can validate your setup by running:

```bash
kubectl version --short
```

You should see output with both a Client Version and Server Version component.

Now that we have our cluster, we’ll install the Marblerun CLI.

## Step 1: Install the CLI

If this is your first time running Marblerun, you will need to download the marblerun command-line interface (CLI) onto your local machine. The CLI will allow you to interact with your Marblerun deployment.

To install the CLI manually, run:

### For the current user

```bash
wget -P ~/.local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun
chmod +x ~/.local/bin/marblerun
```

### Global install (requires root)

```bash
sudo -O /usr/local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun
sudo chmod +x /usr/local/bin/marblerun
```

Once installed, verify the CLI is running correctly with:

```bash
marblerun
```

## Step 2: Install the control-plane onto your cluster

Now that you have the CLI running locally and a cluster that is ready to go, it’s time to install the control plane.

The first step is to install Marblerun's *Coordinator* control-plane. To do this, run:

```bash
marblerun install --simulation
```

In this command, the marblerun install command generates a Kubernetes manifest with all the necessary control plane resources.
The simulation flag tells Marblerun that real SGX hardware might not be present and the SGX-layer should be emulated.

## Step 3: Initialize and verify the Coordinator

After installing the Coordinator we need to verify its integrity and establish trust in a confidential computing sense.
Therefore, we utilize SGX remote attestation and obtain a root certificate for the entire confidential service mesh.

1. Port forward the Coordinator's Client API

```bash
kubectl -n marblerun port-forward svc/coordinator-client-api 25555:25555 --address localhost >/dev/null &
export MARBLERUN=localhost:25555
```

1. Verify the Quote and get the Coordinator's Root-Certificate

```bash
marblerun certificate root $MARBLERUN -o marblerun.crt --insecure
```

We need the `insecure` flag because Marblerun was installed in simulation mode, hence, the Coordinator can't be verified with real SGX attestation.

The certificate can be used for authenticating the Coordinator in future interactions.
It can also be used as a root of trust for [authenticating your confidential applications]({{< ref "docs/features/attestation.md#external-attestation-of-the-app" >}}).

## Step 4: Deploy the demo application

To get a feel for how Marblerun would work for one of your services, you can install a demo application.
The emojivoto application is a standalone Kubernetes application that uses a mix of gRPC and HTTP calls to allow the users to vote on their favorite emojis.
Created as a demo application for the popular [Linkerd](https://linkerd.io) service mesh, we've created a confidential variant that uses TLS for all gRPC and HTTP connections.
Clone the demo application from the [GitHub repository]( https://github.com/edgelesssys/emojivoto.git) by running:

```bash
git clone https://github.com/edgelesssys/emojivoto.git && cd emojivoto
```

Marblerun guarantees that the topology of your distributed app adheres to a Manifest specified in simple JSON.
Marblerun verifies the integrity of services, bootstraps them, and sets up encrypted connections between them.
The emojivoto demo already comes with a [manifest](https://github.com/edgelesssys/emojivoto/blob/main/tools/manifest.json), which you can deploy onto Marblerun by running:

```bash
marblerun manifest set tools/manifest.json $MARBLERUN --insecure
```

You can check that the state of Marblerun changed and is now ready to authenticate your services by running:

```bash
marblerun status
```

Finally, install the demo application onto your cluster.
Please make sure you have installed [Helm v3](https://helm.sh/docs/intro/install/) ("the package manager for Kubernetes").
Install emojivoto into the emojivoto namespace by running:

```bash
helm install -f ./kubernetes/nosgx_values.yaml emojivoto ./kubernetes --create-namespace -n emojivoto
```

## Step 5: Watch it run

You can now check the Marblerun log and see the services being authenticated by the coordinator.

```bash
kubectl -n marblerun logs -f -ledgeless.systems/control-plane-component=coordinator
```

Port forward the front-end web service to access it on your local machine by running:

```bash
sudo kubectl -n emojivoto port-forward svc/web-svc 443:443 --address 0.0.0.0
```

Now visit [https://localhost](https://localhost).
You'll be presented with a certificate warning because your browser does not know Marblerun's root certificate as a root of trust.
You can safely ignore this error for now and proceed to the website.\
Voila! Your emoji votes have never been safer!

## That’s it! :clap:

Congratulations, you’re now a Marblerun user! Here are some suggested next steps:

* Explore how [Marblerun takes care of your secrets]({{< ref "docs/features/secrets-management.md" >}})
* [Add your own service]({{< ref "docs/tasks/add-service.md" >}}) to Marblerun
* Learn more about [Marblerun’s architecture]({{< ref "docs/getting-started/concepts.md" >}})
* Hop into our [Gitter community chat](https://gitter.im/edgeless-systems/community)

Welcome to the Marblerun community!
