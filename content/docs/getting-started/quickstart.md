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

Please also install the Marblerun CLI:
### For the current user
```bash
wget -P ~/.local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun-cli
chmod +x ~/.local/bin/marblerun
```
### Global install (requires root)
```bash
sudo -O /usr/local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun-cli
sudo chmod +x /usr/local/bin/marblerun
```

## Step 1: Install the Coordinator onto the cluster

Install Marblerun's Coordinator using Helm.
Update the hostname with your cluster's FQDN or use localhost for local testing.

Install Marblerun's Coordinator using the CLI.
Update the hostname with your cluster's FQDN or use localhost for local testing.

* For a cluster with SGX support:
    ```bash
    marblerun install --domain=localhost
    ```

* For a cluster without SGX support:

    ```bash
    marblerun install --domain=localhost --simulation
    ```

## Step 2: Pull the demo application

```bash
git clone https://github.com/edgelesssys/emojivoto.git && cd emojivoto
```

## Step 3: Initialize and verify the Coordinator

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

1. Verify the Quote and get the Coordinator's Root-Certificate
    * If you're running on a cluster with nodes that support SGX1+FLC

        ```bash
        marblerun certificate root $MARBLERUN -o marblerun.crt
        ```

    * Otherwise

        ```bash
        marblerun certificate root $MARBLERUN -o marblerun.crt --insecure
        ```

## Step 4: Set the Manifest

    * If you're running on a cluster with nodes that support SGX1+FLC

        ```bash
        marblerun manifest set tools/manifest.json $MARBLERUN
        ```

    * Otherwise

        ```bash
        marblerun manifest set tools/manifest.json $MARBLERUN --insecure
        ```

* If you're running emojivoto on a custom domain, you can set the certificate's CN accordingly

    ```bash
    cat "tools/manifest.json" | sed "s/localhost/<your-domain>/g" > manifest.json
    marblerun manifest set manifest.json $MARBLERUN
    ```

## Step 5: Deploy the demo application

* If you're deploying on a cluster with nodes that support SGX1+FLC (e.g. AKS or minikube + Azure Standard_DC*s)

  ```bash
  helm install -f ./kubernetes/sgx_values.yaml emojivoto ./kubernetes --create-namespace -n emojivoto
  ```

* Otherwise

  ```bash
  helm install -f ./kubernetes/nosgx_values.yaml emojivoto ./kubernetes --create-namespace -n emojivoto
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
