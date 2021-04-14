---
title: "Quickstart"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 1
---

# Quickstart


Set up a Kubernetes cluster and install `kubectl`. Probably the easiest way to get started is to run Kubernetes on your local machine using [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/). Another easy way is to use [Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal), which offers SGX-enabled nodes.

In this guide will show you how to deploy and verify the [Confidential Emoji.voto](https://github.com/edgelesssys/emojivoto) application, an microservice that allows users to vote for their favorite emoji, and tracks votes received on a leaderboard.

Choose either to follow the [First Steps on Minikube]({{< ref "docs/getting-started/quickstart.md#first-steps-on-minikube" >}}) or use a cluster with SGX support (SGX1+FLC) and start with the [First Steps on AKS]({{< ref "docs/getting-started/quickstart.md#first-steps-on-aks" >}}).

# First steps on Minikube

## Step 0: Setup

[Install Helm](https://helm.sh/docs/intro/install/), the "package manager" for Kubernetes. Helm should be at version v3.2.0 or higher. You can check the version of an existing helm installation as following:
```bash
helm version
```

Next, install the Marblerun CLI from our latest binary release:
+ For the current user
    ```bash
    wget -P ~/.local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun-cli
    chmod +x ~/.local/bin/marblerun
    ```
+ Global install (requires root)
    ```bash
    sudo wget -O /usr/local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun-cli
    sudo chmod +x /usr/local/bin/marblerun
    ```

## Step 1: Deploy the Coordinator onto the cluster

Deploy Marblerun's Coordinator to the Minikube cluster using the CLI.

```bash
marblerun install --domain=localhost --simulation
```

Wait for Marblerun to start:

```bash
marblerun check
```

## Step 2: Pull the demo application

```bash
git clone https://github.com/edgelesssys/emojivoto.git && cd emojivoto
```

## Step 3: Initialize and verify the Coordinator

Get the Coordinator's address and set the DNS

```bash
kubectl -n marblerun port-forward svc/coordinator-client-api 4433:4433 --address localhost >/dev/null &
export MARBLERUN=localhost:4433
```

Verify the Quote and get the Coordinator's Root-Certificate. The SGX Quote proofs the integrity of the coordinator pod. Marblerun returns a certificate as result and stores it as marblerun.cert in your current directory. The Certificate is bound to the Quote and can be used for future verification. Since we are not using SGX hardware in this case, the quote is omitted by marblerun.

```bash
marblerun certificate root $MARBLERUN -o marblerun.crt --insecure
```

## Step 4: Set the Manifest

```bash
marblerun manifest set tools/manifest.json $MARBLERUN --insecure
```

## Step 5: Deploy the demo application

```bash
helm install -f ./kubernetes/nosgx_values.yaml emojivoto ./kubernetes --create-namespace -n emojivoto
```

## Step 6: Watch it run

Make the voting frontend reachable for with port-forwarding:

```bash
sudo kubectl -n emojivoto port-forward svc/web-svc 443:443 --address 0.0.0.0
```

Install Marblerun-Certificate in your browser
* **Warning** Be careful when adding certificates to your browser. We only do this temporarily for the sake of this demo. Make sure you don't use your browser for other activities in the meanwhile and remove the certificate afterward.
* Chrome:
    * Go to <chrome://settings/security>
    * Go to `"Manage certificates" > "Import..."`
    * Follow the "Certificate Import Wizard" and import the `marblerun.crt` of the previous step as a "Personal" certificate
* Firefox:
    * Go to <about:preferences#privacy>
    * Go to `Certificates: View Certificates > Authorities`
    * Go to `Import...` and select the `marblerun.crt` of the previous step

Browse to [https://localhost](https://localhost).


# First Steps on AKS

## Step 0: Setup

[Install Helm](https://helm.sh/docs/intro/install/), the "package manager" for Kubernetes. Helm should be at version v3.2.0 or higher. You can check the version of an existing helm installation as following:
```bash
helm version
```

Next, install the Marblerun CLI from our latest binary release:
### For the current user
```bash
wget -P ~/.local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun-cli
chmod +x ~/.local/bin/marblerun
```
### Global install (requires root)
```bash
sudo wget -O /usr/local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun-cli
sudo chmod +x /usr/local/bin/marblerun
```

### Verify the cluster supports SGX
```bash
marblerun precheck
```
If the command reports your cluster does not support SGX verify the cluster was deployed correctly as described in [this guide](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-aks-get-started) or consider following the [First Steps on Minikube]({{< ref "docs/getting-started/quickstart.md#first-steps-on-minikube" >}}).

## Step 1: Deploy the Coordinator onto the cluster

Deploy Marblerun's Coordinator to the Kubernetes cluster using the CLI. Update the domain parameter with your cluster's domain name.

```bash
marblerun install --domain=mycluster.uksouth.cloudapp.azure.com
```

Wait for Marblerun to start:

```bash
marblerun check
```

## Step 2: Pull the demo application

```bash
git clone https://github.com/edgelesssys/emojivoto.git && cd emojivoto
```

## Step 3: Initialize and verify the Coordinator

Get the Coordinator's address and set the DNS. Check our docs on [how to expose the Client-API]({{< ref "docs/tasks/deploy.md#accesing-the-client-api" >}})

```bash
export MARBLERUN=mycluster.uksouth.cloudapp.azure.com
```

Verify the Quote and get the Coordinator's Root-Certificate. The SGX Quote proofs the integrity of the coordinator pod. Marblerun returns a certificate as result and stores it as marblerun.cert in your current directory. The Certificate is bound to the Quote and can be used for future verification.

```bash
marblerun certificate root $MARBLERUN -o marblerun.crt
```

## Step 4: Set the Manifest

Set the certificate's CN accordingly and hand the manifest to the Marblerun Coordinator.

```bash
cat "tools/manifest.json" | sed "s/localhost/<your-domain>/g" > manifest.json
marblerun manifest set manifest.json $MARBLERUN
```

## Step 5: Deploy the demo application

```bash
helm install -f ./kubernetes/sgx_values.yaml emojivoto ./kubernetes --create-namespace -n emojivoto
```

## Step 6: Watch it run

* Exposing your service
    * You need to expose the `web-svc` in the `emojivoto` namespace. This works similar to [how we expose the client-API]({{< ref "docs/tasks/deploy.md#accesing-the-client-api" >}})
    * Get the public IP with: `kubectl -n emojivoto get svc web-svc -o wide`
    * If you're using ingress/gateway-controllers make sure you enable [SNI-passthrough]({{< ref "docs/tasks/deploy.md#ingressgateway-configuration" >}})

* Install Marblerun-Certificate in your browser
    * **Warning** Be careful when adding certificates to your browser. We only do this temporarily for the sake of this demo. Make sure you don't use your browser for other activities in the meanwhile and remove the certificate afterward.
    * Chrome:
        * Go to <chrome://settings/security>
        * Go to `"Manage certificates" > "Import..."`
        * Follow the "Certificate Import Wizard" and import the `marblerun.crt` of the previous step as a "Personal" certificate
   * Firefox:
        * Go to <about:preferences#privacy>
        * Go to `Certificates: View Certificates > Authorities`
        * Go to `Import...` and select the `marblerun.crt` of the previous step

* Browse to [https://your-clusters-domain:port](#).
