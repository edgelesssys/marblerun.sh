---
title: "CLI"
draft: false
weight: 5
---

# Command Line Interface (CLI)

We provide a command-line interface (CLI) for Marblerun.
This CLI allows you to install Marblerun on your cluster and interacts with the control plane through the Client API for all administrative tasks in the service mesh.

To install the Marblerun CLI on your machine you can use our pre-built binaries.
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

To build the Marblerun CLI, [Edgeless RT](https://github.com/edgelesssys/edgelessrt) needs to be installed on your machine.

```bash
go build -o marblerun github.com/edgelesssys/marblerun/cli
```

To list all available commands, either run `marblerun` with no commands or execute `marblerun help`
The output is the following:

```bash
Usage:
  marblerun [command]

Available Commands:
  certificate Retrieves the certificate of the Marblerun coordinator
  help        Help about any command
  install     Installs marblerun on a kubernetes cluster
  manifest    Manages manifest for the Marblerun coordinator
  namespace   Manages namespaces associated with Marblerun installations
  recover     Recovers the Marblerun coordinator from a sealed state
  status      Gives information about the status of the marblerun Coordinator

Flags:
  -h, --help   help for marblerun

Use "marblerun [command] --help" for more information about a command.
```

## **`install`**

Install Marblerun on your Kubernetes cluster.
This command will add Marblerun to your local helm repository if it is not present yet, optionally you can provide a path to your own helm chart.

**Usage**

```bash
marblerun install [flags]
```

**Flags**

{{<table "table table-striped table-bordered">}}
| Name, shorthand        | Default       | Description                                                    |
|:-----------------------|:--------------|:---------------------------------------------------------------|
| --client-server-port   | 25555         | Set the client server port. Needs to be configured to the same <br> port as in your client tool stack |
| --domain               | localhost     | Sets the CNAME for the coordinator certificate                 |
| --help, -h             |               | help for install                                               |
| --marblerun-chart-path |               | Path to marblerun helm chart                                   |
| --mesh-sever-port      | 25554         | Set the mesh server port. Needs to be configured to the same <br> port as in the data-plane marbles |
| --no-sgx-device-plugin |               | Disables the installation of an sgx device plugin              |
| --simulation           |               | Set Marblerun to start in simulation mode, needed when not <br> running on an SGX enabled cluster |
{{</table>}}

**Examples**

* Install Marblerun on a cluster with SGX Support

    ```bash
    marblerun install --domain=mycluster.uksouth.cloudapp.azure.com
    ```

  The output is similar to the following:

    ```bash
    Did not find marblerun helm repository on system, adding now...
    edgeless has been added to your helm repositories
    Marblerun installed successfully
    ```

* Install Marblerun on a cluster without SGX Support (simulation mode)

    ```bash
    marblerun install --simulation
    ```

  The output is similar to the following:

  ```bash
  Marblerun installed successfully
  ```


## **`status`**

Checks on the current status of the coordinator.

**Usage**

```bash
marblerun status <IP:PORT> [flags]
```

**Flags**

{{<table "table table-striped table-bordered">}}
| Name, shorthand        | Default       | Description                                                    |
|------------------------|---------------|----------------------------------------------------------------|
| --era-config           |               | Path to remote attestation config file in json format, if none <br> provided the newest configuration will be loaded from github |
| --help, -h             |               | help for status                                                |
| --insecure, -i         |               | Set to skip quote verification, needed when running in <br> simulation mode |
{{</table>}}

**Examples**

```bash
marblerun status $MARBLERUN
```

The output is similar to the following:

```bash
No era config file specified, getting latest config from github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
Got latest config
2: Coordinator is ready to accept a manifest.
```


## **`manifest`**

Set or update a manifest, or retrieve the signature of the manifest in place.

**Flags**
These flags apply to all sub commands of manifest

{{<table "table table-striped table-bordered">}}
| Name, shorthand        | Default       | Description                                                    |
|------------------------|---------------|----------------------------------------------------------------|
| --era-config           |               | Path to remote attestation config file in json format, if none <br> provided the newest configuration will be loaded from github |
| --help, -h             |               | help for manifest                                              |
| --insecure, -i         |               | simulation mode                                                |
{{</table>}}

* ### **`set`**

  Uploads a manifest in json format to the Marblerun coordinator.
  If a recovery key was set in the manifest, a recovery secret will be sent back.

  **Usage**

  ```bash
  marblerun manifest set <manifest.json> <IP:PORT> [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand        | Default       | Description                                                    |
  |------------------------|---------------|----------------------------------------------------------------|
  | --recovery-data, -r    |               | File to write recovery data to, print to stdout if not set     |
  {{</table>}}

  **Examples**

  ```bash
  marblerun manifest set manifest.json $MARBLERUN --recovery-data=recovery-secret.json --era-config=era.json
  ```

  The output is similar to the following:

  ```bash
  Successfully verified coordinator, now uploading manifest
  Manifest successfully set, recovery data saved to: recovery-secret.json
  ```


* ### **`update`**

  Update a manifest by uploading an update manifest to the Marblerun coordinator.
  The original manifest has to define one or multiple Admins who are allowed to update the manifest.
  For more information see [Update]({{< ref "docs/tasks/update-manifest.md" >}})

  **Usage**

  ```bash
  marblerun manifest update <manifest.json> <IP:PORT> --cert=admin-cert.pem --key=admin-key.pem [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand        | Default       | Description                                                    |
  |------------------------|---------------|----------------------------------------------------------------|
  | --cert, -c             |               | PEM encoded admin certificate file (required)                  |
  | --key, -k              |               | PEM encoded admin key file (required)                          |
  {{</table>}}

  **Examples**

  ```bash
  marblerun manifest update update-manifest.json $MARBLERUN --cert=admin-cert.pem --key=admin-key.pem --era-config=era.json
  ```

  The ouput is the following:

  ```bash
  Successfully verified coordinator, now uploading manifest
  Manifest successfully updated
  ```

* ### **`get`**

  Retrieves the signature of an uploaded manifest. This allows a user to verify what manifest is running on the coordinator.

  **Usage**

  ```bash
  marblerun manifest get <IP:PORT> [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand        | Default       | Description                                                    |
  |------------------------|---------------|----------------------------------------------------------------|
  | --output, -o           | signature.json | Define file to write to                                        |
  {{</table>}}

  **Examples**

  ```bash
  marblerun manifest get $MARBLERUN --output=manifest-signature.json --era-config=era.json
  ```

  The output is the following:

  ```bash
  Successfully verified coordinator, now requesting manifest signature
  Manifest written to: manifest-signature.json
  ```

## **`recover`**

Recover the Marblerun coordinator from a sealed state by uploading a recovery key.
For more information about coordinator recovery see [Recovery]({{< ref "docs/tasks/recover-coordinator.md" >}})

**Usage**

```bash
marblerun recover <IP:PORT> <recovery_key_decrypted> [flags]
```

**Flags**

{{<table "table table-striped table-bordered">}}
| Name, shorthand        | Default       | Description                                                    |
|------------------------|---------------|----------------------------------------------------------------|
| --era-config           |               | Path to remote attestation config file in json format, if none <br> provided the newest configuration will be loaded from github |
| --help, -h             |               | help for recover                                               |
| --insecure, -i         |               | Set to skip quote verification, needed when running in <br> simulation mode |
{{</table>}}

**Examples**

```bash
marblerun recover $MARBLERUN recovery_key_decrypted --era-config=era.json
```

The output is similar to the following:

```bash
Successfully verified coordinator, now uploading key
Successfully uploaded recovery key and unsealed the Marblerun coordinator
```


## **`certificate`**

Get the root and/or intermediate certificates of the Marblerun coordinator.

**Flags**
These flags apply to all sub commands of certificate

{{<table "table table-striped table-bordered">}}
| Name, shorthand        | Default       | Description                                                    |
|------------------------|---------------|----------------------------------------------------------------|
| --era-config           |               | Path to remote attestation config file in json format, if none <br> provided the newest configuration will be loaded from github |
| --help, -h             |               | help for certificate                                           |
| --insecure, -i         |               | simulation mode                                                |
| --output, -o           |               | File to save the certificate to                                |
{{</table>}}

* ### **`root`**

  Gets the root certificate of the Marblerun coordinator.

  **Usage**

  ```bash
  marblerun certificate root <IP:PORT> [flags]
  ```

* ### **`intermediate`**

  Gets the intermediate certificate of the Marblerun coordinator.

  **Usage**

  ```bash
  marblerun certificate intermediate <IP:PORT> [flags]
  ```

* ### **`chain`**

  Gets the certificate chain of the Marblerun coordinator.

  **Usage**

  ```bash
  marblerun certificate chain <IP:PORT> [flags]
  ```


## **`namespace`**

Add namespaces Marblerun.
If the auto-injection feature is enabled. All new pods in those namespaces will get their Marblerun configuration automatically injected.


* ### **`add`**

  Add a namespace to the Marblerun mesh by creating a new label

  **Usage**

  ```bash
  marblerun namespace add NAMESPACE ... [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand        | Default       | Description                                                    |
  |------------------------|---------------|----------------------------------------------------------------|
  | --inject-sgx           |               | Set to enable automatic injection of SGX tolerations for <br> namespace |
  {{</table>}}

  **Examples**

  ```bash
  marblerun namespace add default testspace
  ```

  The output is the following:

  ```bash
  Added namespace [default] to Marblerun mesh
  Added namespace [testspace] to Marblerun mesh
  ```

* ### **`remove`**

  Remove a namespace from the Marblerun mesh

  **Usage**

  ```bash
  marblerun namespace remove NAMESPACE [flags]
  ```

  **Examples**

  ```bash
  marblerun namespace remove default
  ```

  The output is the following:

  ```bash
  Namespace [default] succesfully removed from the Marblerun mesh
  ```

* ### **`list`**

  List all namespaces currently associated with the Marblerun mesh

  **Usage**

  ```bash
  marblerun namespace list
  ```

  **Examples**

  ```bash
  marblerun namespace list
  ```

  The output is the following:

  ```bash
  testspace
  ```
