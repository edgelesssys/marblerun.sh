---
title: "Quickstart"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 1
---

# Quickstart

## Step 0: Setup
Set up a Kubernetes cluster and install `kubectl`. Probably the easiest way to get started is to run Kubernetes on your local machine using [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/). Another easy way is to use [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal), which offers SGX-enabled nodes. 

Please also install [Helm](https://helm.sh/docs/intro/install/) ("the package manager for Kubernetes").

## Step 1: Install the Coordinator onto the cluster

Add the Edgeless Systems chart repository to Helm.

```bash
helm repo add edgeless https://helm.edgeless.systems
helm repo update
```

Install Marblerun's Coordinator using Helm.
Update the hostname with your cluster's FQDN or use localhost for local testing.

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

## Step 2: Pull the demo application

```bash
git clone https://github.com/edgelesssys/emojivoto.git && cd emojivoto
```

## Step 3: Initialize and verify the Coordinator

1. Pull the remote attestation configuration

    ```bash
    wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
    ```

1. Get the Coordinator's address and set the DNS

    * If you're running on AKS:
        * Check our docs on [how to set the DNS for the Client-API]({{< ref "docs/tasks/deploy.md#dns-for-the-client-api-on-azure-kubernetes-service-aks" >}})

            ```bash
            export MARBLERUN=mycluster.uksouth.cloudapp.azure.com
            ```

    * If you're running on minikube

        ```bash
        kubectl -n marblerun port-forward svc/coordinator-client-api 25555:25555 --address localhost >/dev/null &
        export MARBLERUN=localhost:25555
        ```

1. Install the Edgeless Remote Attestation Tool
    1. Check [requirements](https://github.com/edgelesssys/era#requirements)
    2. See [install](https://github.com/edgelesssys/era#install)

1. Verify the Quote and get the Coordinator's Root-Certificate
    * If you're running on a cluster with nodes that support SGX1+FLC

        ```bash
        era -c coordinator-era.json -h $MARBLERUN -o marblerun.crt
        ```

    * Otherwise

        ```bash
        era -skip-quote -c coordinator-era.json -h $MARBLERUN -o marblerun.crt
        ```

## Step 4: Set the Manifest

```bash
curl --silent --cacert marblerun.crt -X POST -H  "Content-Type: application/json" --data-binary @tools/manifest.json "https://$MARBLERUN/manifest"
```

## Step 5: Deploy the demo application

* If you're deploying on a cluster with nodes that support SGX1+FLC (e.g. AKS or minikube + Azure Standard_DC*s)

  ```bash
  helm install -f ./kubernetes/sgx_values.yaml emojivoto ./kubernetes -n emojivoto
  ```

* Otherwise

  ```bash
  helm install -f ./kubernetes/nosgx_values.yaml emojivoto ./kubernetes -n emojivoto
  ```

## Step 6: Watch it run

* If you're running on AKS
    * You need to expose the `web-svc` in the `emojivoto` namespace. This works similar to [how we expose the client-API]({{< ref "docs/tasks/deploy.md#dns-for-the-client-api-on-azure-kubernetes-service-aks" >}})
    * Get the public IP with: `kubectl -n emojivoto get svc web-svc -o wide`
    * If you're using ingress/gateway-controllers make sure you enable [SNI-passthrough]({{< ref "docs/tasks/deploy.md#ingressgateway-configuration" >}})
* If you're running on minikube

    ```bash
    sudo kubectl -n emojivoto port-forward svc/web-svc 443:443 --address 0.0.0.0
    ```

* Install Marblerun-Certificate in your browser
    * **Warning** Be careful when adding certificates to your browser. We only do this temporarily for the sake of this demo. Make sure you don't use your browser for other activities in the meanwhile and remove the certificate afterward.
    * Chrome:
        * Go to <chrome://settings/security>
        * Go to `"Manage certificates" > "Import..."`
        * Follow the "Certificate Import Wizard" and import the `marblerun.crt` of the previous step as a "Personal" certificate
    * Firefox:
        * Go to `Tools > Options > Advanced > Certificates: View Certificates`
        * Go to `Import...` and select the `marblerun.crt` of the previous step

* Browse to [https://localhost](https://localhost) or [https://your-clusters-fqdn:port](#) depending on your type of deployment.

* Notes on DNS: If your running emojivoto on a remote machine you can add the machine's DNS name to the emojivoto certificate (e.g. `emojivoto.example.org`):

  * Open the `kubernetes/sgx_values.yaml` or `kubernetes/nosgx_values.yaml` file depending on your type of deployment

  * Add your DNS name to the `hosts` field:

    * `hosts: "emojivoto.example.org,localhost,web-svc,web-svc.emojivoto,web-svc.emojivoto.svc.cluster.local"`

  * You need to apply your changes with:

    * If your using `kubernetes/sgx_values.yaml` for your deployment:

        ```bash
        helm upgrade -f ./kubernetes/sgx_values.yaml emojivoto ./kubernetes -n emojivoto
        ```

    * Otherwise:

        ```bash
        helm upgrade -f ./kubernetes/nosgx_values.yaml emojivoto ./kubernetes -n emojivoto
        ```
