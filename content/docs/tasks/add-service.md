---
title: "Adding a service"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 3
---

# Adding a service

Adding a service to your application requires three steps, which are described in the following.

## **Step 1:** Get your service ready for Marblerun

To get your service ready for Marblerun, you possibly need to adapt its code slightly and you need to rebuild it. Details are given in the following steps 1.1 and 1.2. *Note that we are working on making these unnecessary in the future - at least for services written in Go.*

### **Step 1.1:** Make your service use the provided TLS credentials

Quick refresher: Marblerun's Coordinator issues TLS credentials for each verified Marble (i.e., a service running in a secure enclave) as is described [here]({{< ref "docs/features/secrets-management.md#tls-credentials" >}}).

The TLS X.509 certificate and the corresponding private key can be securely passed to a service through files, environments variables, or commandline arguments. This is defined in the Manifest as is described [here]({{< ref "docs/tasks/define-manifest.md#manifestmarbles" >}}).

For now, you just need to make sure that your service reads the certificate and the private key from arbitrary paths, environment variables, or commandline arguments, e.g., the file `/tmp/mycert.cert` or the environment variable `MY_PRIVATE_KEY`, and uses them at runtime for internal and external connections. If you're lucky, your service already does this and you don't need to change a thing in the code.

For services written in Go, we provide a convenience package called `github.com/edgelesssys/ertgolib/marble`. With it, a service can automatically get and use its Marblerun TLS credentials. The following gives an example.
```Go
func main() {
    serverCfg, err := marble.GetTLSConfig(false)
    if err != nil {
        log.Fatalf("Failed to retrieve server TLS config from ertgolib")
    }
    serverCreds := credentials.NewTLS(serverCfg)
    // use serverCreds, e.g., to create an HTTPS server
}
```

### **Step 1.2:** Re-compile/build your service for Marblerun

Finally, you need to re-build your service for the enclave environment and include/link Marblerun-specific code. Please follow the build instructions for Go provided [here](https://github.com/edgelesssys/marblerun/blob/master/samples/helloworld) or the build instructions for C++ provided [here](https://github.com/edgelesssys/marblerun/blob/master/samples/helloc%2B%2B).

## **Step 2:** Define your service in the Manifest

Now that your service is ready, you need to make two types of entries in the Manifest regarding its properties and parameters.

### **Step 2.1:** Define the enclave software-package

As is described in more detail [here]({{< ref "docs/tasks/define-manifest.md#manifestpackages" >}}), the Manifest contains a section `Packages`, in which allowed enclave software-packages are defined.

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

Now you can define with which parameters (i.e., files, environments variables, and command line arguments) your service is allowed to run. This is done in the `Marbles` section of the Manifest as is described [here]({{< ref "docs/tasks/define-manifest.md#manifestmarbles" >}}). As discussed in [Step #1.1](#step-11-make-your-service-use-the-provided-tls-credentials), you need to make sure that the TLS credentials for your service (i.e., `Marblerun.MarbleCert.Cert` and `Marblerun.MarbleCert.Private`) are injected such that your service will find them at runtime. If your service is written in Go and you're using the `marble` package, there is no need to inject these explicitly.

## **Step 3:** Start your service

When you start your service, you need to pass in a couple of configuration parameters through environment variables. Here is an example:

```bash
EDG_MARBLE_COORDINATOR_ADDR=coordinator-mesh-api.marblerun:25554 EDG_MARBLE_TYPE=mymarble EDG_MARBLE_UUID_FILE=$PWD/uuid EDG_MARBLE_DNS_NAMES=localhost,myservice erthost enclave.signed
```

`erthost` is the generic host for Marbles, which will load your `enclave.signed`. The environment variables have the following purposes.

* `EDG_MARBLE_COORDINATOR_ADDR` is the network address of the Coordinator's API for Marbles. When you deploy the Coordinator using our Helm repository as is described [here]({{< ref "docs/tasks/deploy.md" >}}), the default address is `coordinator-mesh-api.marblerun:25554`.

* `EDG_MARBLE_TYPE` needs to reference one entry from your Manifest's `Marbles` section.

* `EDG_MARBLE_UUID_FILE` is the local file path where the Marble stores its UUID. Every instance of a Marble has its unique and public UUID. The file is needed to allow a Marble to restart under its UUID.

* `EDG_MARBLE_DNS_NAMES` is the list of DNS names the Coordinator will issue the Marble's certificate for.

Typically, you will define these in a Kubernetes manifest or a Helm chart, for example:

```yaml
spec:
    containers:
    - env:
    - name: EDG_MARBLE_COORDINATOR_ADDR
        value: coordinator-mesh-api.marblerun:25554
    - name: EDG_MARBLE_TYPE
        value: mymarble
    - name: EDG_MARBLE_DNS_NAMES
        value: "localhost,myservice"
    - name: EDG_MARBLE_UUID_FILE
        value: "$PWD/uuid"
```

Refer to our [emojivoto](https://github.com/edgelesssys/emojivoto) app for Helm chart examples.
