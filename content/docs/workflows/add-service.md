---
title: "Adding a service"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 4
---

# Adding a service

Adding a service to your application requires three steps, which are described in the following.

## **Step 1:** Get your service ready for Marblerun

To get your service ready for Marblerun, you need to rebuild it with one of the supported [runtimes]({{< ref "docs/features/runtimes.md" >}}):
* [EGo]({{< ref "docs/building-services/ego.md" >}})
* [Edgeless RT](https://github.com/edgelesssys/marblerun/blob/master/samples/helloc%2B%2B)
* [Graphene]({{< ref "docs/building-services/graphene.md" >}})

### Make your service use the provided TLS credentials

Skip this step, when using EGo with TTLS.

Quick refresher: Marblerun's Coordinator issues TLS credentials for each verified Marble (i.e., a service running in a secure enclave) as is described in our [secrets management chapter]({{< ref "docs/features/secrets-management.md#tls-credentials" >}}).

The TLS X.509 certificate and the corresponding private key can be securely passed to a service through files, environments variables, or commandline arguments. This is defined in the Manifest as is described in our [writing a manifest hands-on]({{< ref "docs/workflows/define-manifest.md#manifestmarbles" >}}).

For now, you just need to make sure that your service reads the certificate and the private key from arbitrary paths, environment variables, or commandline arguments, e.g., the file `/tmp/mycert.cert` or the environment variable `MY_PRIVATE_KEY`, and uses them at runtime for internal and external connections. If you're lucky, your service already does this and you don't need to change a thing in the code.

## **Step 2:** Define your service in the Manifest

Now that your service is ready, you need to make two types of entries in the Manifest regarding its properties and parameters.

### **Step 2.1:** Define the enclave software-package

As is described in more detail in our [writing a manifest hands-on]({{< ref "docs/workflows/define-manifest.md#manifestpackages" >}}), the Manifest contains a section `Packages`, in which allowed enclave software-packages are defined.

To add an entry for your service, run the `oesign` tool on the enclave file you built in the previous step as follows. (`oesign` is installed with [Edgeless RT](https://github.com/edgelesssys/edgelessrt).)

```bash
oesign eradump -e enclave.signed
```

The tool's output will look like the following.

```json
{
    "UniqueID": "6b2822ac2585040d4b9397675d54977a71ef292ab5b3c0a6acceca26074ae585",
    "SignerID": "5826218dbe96de0d7b3b1ccf70ece51457e71e886a3d4c1f18b27576d22cdc74",
    "SecurityVersion": 1,
    "ProductID": 3
}
```

Use `UniqueID` (i.e., `MRENCLAVE` in Intel SGX speak) or the triplet of `SignerID` (i.e., `MRSIGNER`), `SecurityVersion`, and `ProductID` to add an entry in the `Packages` section.

### **Step 2.2:** Define the parameters

Now you can define with which parameters (i.e., files, environments variables, and command line arguments) your service is allowed to run. This is done in the `Marbles` section of the Manifest as is described in our [writing a manifest hands-on]({{< ref "docs/workflows/define-manifest.md#manifestmarbles" >}}). When using EGo, define all TTLS connections as described in the [manifest hands-on]({{< ref "docs/workflows/define-manifest.md#manifesttls" >}}).

Otherwise, as discussed in [Step #1.1](#step-11-make-your-service-use-the-provided-tls-credentials), you need to make sure that the TLS credentials for your service (i.e., `Marblerun.MarbleCert.Cert` and `Marblerun.MarbleCert.Private`) are injected such that your service will find them at runtime. If your service is written in Go and you're using the `marble` package, there is no need to inject these explicitly.

## **Step 3:** Start your service

When you start your service, you need to pass in a couple of configuration parameters through environment variables. Here is an example:

```bash
EDG_MARBLE_COORDINATOR_ADDR=coordinator-mesh-api.marblerun:2001 EDG_MARBLE_TYPE=mymarble EDG_MARBLE_UUID_FILE=$PWD/uuid EDG_MARBLE_DNS_NAMES=localhost,myservice erthost enclave.signed
```

`erthost` is the generic host for Marbles, which will load your `enclave.signed`. The environment variables have the following purposes.

* `EDG_MARBLE_COORDINATOR_ADDR` is the network address of the Coordinator's API for Marbles. When you deploy the Coordinator using our Helm repository as is described in our [deploying Marblerun hands-on]({{< ref "docs/deployment" >}}), the default address is `coordinator-mesh-api.marblerun:2001`.

* `EDG_MARBLE_TYPE` needs to reference one entry from your Manifest's `Marbles` section.

* `EDG_MARBLE_UUID_FILE` is the local file path where the Marble stores its UUID. Every instance of a Marble has its unique and public UUID. The file is needed to allow a Marble to restart under its UUID.

* `EDG_MARBLE_DNS_NAMES` is the list of DNS names the Coordinator will issue the Marble's certificate for.

## **Step 4:** Deploy your service with Kubernetes

Typically, you'll write a Kubernetes resource definition for your service, which you'll deploy with the Kubernetes CLI, Helm, or similar tools.

For your services to take advantage of Marblerun, they need to be "added to the mesh" by having the data plane configuration injected into their pods.
This is typically done by labeling the namespace, deployment, or pod with the `marblerun/inject=enabled` Kubernetes label.
This label triggers automatic configuration injection when the resources are created. (See the [auto injection page]({{< ref "docs/features/auto-injection.md" >}}) for more on how this works.)
Alternatively, you can enable a namespace for auto-injection using the Marblerun CLI:

```bash
marblerun namespace add NAMESPACE [--no-sgx-injection]
```

In order for our injection service to know which type of Marble your service corresponds to, you also need to add the `marblerun/marbletype` Kubernetes label.
An example for a Marble of type `web` could look like this:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: emojivoto
  labels:
    app.kubernetes.io/name: web
    app.kubernetes.io/part-of: emojivoto
    app.kubernetes.io/version: v1
    marblerun/inject: enabled
    marblerun/marbletype: web
```

This will result in the following configuration being injected when your resources are created:

```yaml
spec:
    containers:
    - env:
    - name: EDG_MARBLE_COORDINATOR_ADDR
        value: coordinator-mesh-api.marblerun:2001
    - name: EDG_MARBLE_TYPE
        value: web
    - name: EDG_MARBLE_DNS_NAMES
        value: "web,web.emojivoto,web.emojivoto.svc.cluster.local"
    - name: EDG_MARBLE_UUID_FILE
        value: "$PWD/uuid"
```

Refer to our [emojivoto](https://github.com/edgelesssys/emojivoto) app for complete Helm chart examples.
