---
title: "Deploying Marblerun"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 1
---

# Deploying Marblerun

This article assumes that you have an existing Kubernetes cluster. Currently, there are several providers offering confidential nodes on CPUs with SGX support:

* [Azure Kubernetes Services](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-aks-overview) confidential nodes on [DCv2 VMs](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-computing-enclaves)
    * An AKS cluster can be created using the [Azure CLI](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough) or the [Azure portal](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal)
* [Alibaba Cloud](https://www.alibabacloud.com/help/doc-detail/108507.htm) ECS Bare Metal Instances
* [IBM Cloud](https://cloud.ibm.com/docs/bare-metal?topic=bare-metal-bm-server-provision-sgx) Bare Metal Servers
* [Equinix](https://metal.equinix.com/product/features/) Bare Metal Servers
* Alternatively, you can deploy the steps with [minikube](https://minikube.sigs.k8s.io/docs/start/)

{{<note>}}
A working SGX DCAP environment is required for Marblerun to work. If you're not running in Azure, you'll likely need to set up your environment according to this [guide](https://software.intel.com/content/www/us/en/develop/articles/intel-software-guard-extensions-data-center-attestation-primitives-quick-install-guide.html). Alternatively, for testing, you can install Marblerun in simulation mode with `--simulation`.
{{</note>}}

## Install with the Marblerun CLI

You can also install Marblerun using the command line interface:

* For a cluster with SGX support:

    ```bash
    marblerun install --domain=mycluster.uksouth.cloudapp.azure.com
    ```

* For a cluster without SGX support:

    ```bash
    marblerun install --domain=mycluster.uksouth.cloudapp.azure.com --simulation
    ```

## Install with Helm

Make sure that you are using the latest release of Helm and have access to the Marblerun Helm repositories. For upgrade instructions, see the [Helm install docs](https://docs.helm.sh/using_helm/#installing-helm). For more information on configuring and using Helm, see [Install applications with Helm in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-helm).

### Adding Marblerun's Helm repository

```bash
helm repo add edgeless https://helm.edgeless.systems/stable
helm repo update
```

### Installing the chart

Update the hostname with your cluster's FQDN.

* For a cluster with SGX support:

    ```bash
    helm install marblerun-coordinator edgeless/marblerun-coordinator \
        --create-namespace \
        -n marblerun \
        --set coordinator.hostname=mycluster.uksouth.cloudapp.azure.com
    ```

* For a cluster without SGX support:

    ```bash
    helm install marblerun-coordinator edgeless/marblerun-coordinator \
        --create-namespace \
        -n marblerun \
        --set coordinator.resources=null \
        --set coordinator.simulation=1 \
        --set tolerations=null \
        --set coordinator.hostname=mycluster.uksouth.cloudapp.azure.com
    ```

## Accessing the client API

The coordinator creates a [`LoadBalancer`](https://kubernetes.io/docs/concepts/services-networking/service/#loadbalancer) service called `coordinator-client-api` exposing the client API on the default port 25555.
Depending on your cloud provider you can provision a LoadBalancer that exposes this service to the outside world or you deploy an Ingress Gateway forwarding the traffic.
If you are running with Minikube you can expose this service to localhost with `kubectl -n marblerun port-forward svc/coordinator-client-api 25555:25555 --address localhost`.

## Ingress/Gateway configuration

If you're using an ingress-controller or gateway for managing access to the coordinator-client-api service make sure you're enabling SNI for your TLS connections.

* For the NGINX ingress controller add the [`nginx.ingress.kubernetes.io/ssl-passthrough`](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#ssl-passthrough) annotation.
* For Istio Gateways set the [tls-mode PASSTHROUGH](https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-sni-passthrough/#configure-an-ingress-gateway)
