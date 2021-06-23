---
title: "CLI"
draft: false
weight: 2
---

# Command Line Interface (CLI)

We provide a command-line interface (CLI) for Marblerun.
This CLI allows you to install Marblerun on your cluster and interacts with the control plane through the Client API for all administrative tasks in the service mesh.

{{< toc >}}
## Installation

To install the Marblerun CLI on your machine you can use our pre-built binaries.

**For the current user**

```bash
wget -P ~/.local/bin https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun
chmod +x ~/.local/bin/marblerun
```

**Global install (requires root)**

```bash
sudo wget -O /usr/local/bin/marblerun https://github.com/edgelesssys/marblerun/releases/latest/download/marblerun
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
  certificate      Retrieves the certificate of the Marblerun coordinator
  check            Check the status of Marbleruns control plane
  completion       Output script for specified shell to enable autocompletion
  graphene-prepare Modifies a Graphene manifest for use with Marblerun
  help             Help about any command
  install          Installs marblerun on a kubernetes cluster
  manifest         Manages manifest for the Marblerun coordinator
  namespace        Manages namespaces associated with Marblerun installations
  precheck         Check if your kubernetes cluster supports SGX
  recover          Recovers the Marblerun coordinator from a sealed state
  secret           Manages secrets for the Marblerun coordinator
  status           Gives information about the status of the marblerun Coordinator
  uninstall        Removes Marblerun from a kubernetes cluster
  version          Display version of this CLI and (if running) the Marblerun coordinator

Flags:
  -h, --help   help for marblerun

Use "marblerun [command] --help" for more information about a command.
```
## Command `certificate`

Get the root and/or intermediate certificates of the Marblerun coordinator.

**Flags**
These flags apply to all `certificate` subcommands

{{<table "table table-striped table-bordered">}}
| Name, shorthand | Default | Description                                                                                                                      |
| --------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| --era-config    |         | Path to remote attestation config file in json format, if none <br> provided the newest configuration will be loaded from github |
| --help, -h      |         | help for certificate                                                                                                             |
| --insecure, -i  |         | simulation mode                                                                                                                  |
| --output, -o    |         | File to save the certificate to                                                                                                  |
{{</table>}}

* ### `root`

  Gets the root certificate of the Marblerun coordinator.

  **Usage**

  ```bash
  marblerun certificate root <IP:PORT> [flags]
  ```

* ### `intermediate`

  Gets the intermediate certificate of the Marblerun coordinator.

  **Usage**

  ```bash
  marblerun certificate intermediate <IP:PORT> [flags]
  ```

* ### `chain`

  Gets the certificate chain of the Marblerun coordinator.

  **Usage**

  ```bash
  marblerun certificate chain <IP:PORT> [flags]
  ```

## Command `check`

  Check the status of Marbleruns control plane.
  This command will check if the Marblerun coordinator and/or the Marblerun webhook are deployed on a Kubernetes cluster and wait until all replicas of the deployment have the `available` status.

  **Usage**

  ```bash
  marblerun check
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand | Default | Description                             |
  | --------------- | ------- | --------------------------------------- |
  | --timeout       | 60      | Time to wait before aborting in seconds |
  {{</table>}}


## Command `completion`
Generate a shell script to enable autocompletion for `marblerun` commands.
Supported shells are:
* `bash`:
  * To enable completion run:
    ```bash
    source <(marblerun completion bash)
    ```

* `zsh`:
  * If completion is not already enabled you need to enable it first:
    ```bash
    echo "autoload -U compinit; compinit" >> ~/.zshrc
    ```
  * Enable completion for `marblerun`:
    ```bash
    marblerun completion zsh > "${fpath[1]}/_marblerun"
    ```


Once enabled, command completion is just one keypress away:\
  `marblerun ma`+<kbd>Tab</kbd> completes to:\
  `marblerun manifest`


## Command `graphene-prepare`
This command helps you if you want to add Graphene-based services to your Marblerun service mesh.
It prepares your Graphene project to be used as a Marble by replacing the original entrypoint of your application with the bootstrapping Marble premain process which eventually spawns your application.
Given your [Graphene manifest template](https://graphene.readthedocs.io/en/latest/manifest-syntax.html), it will suggest the required adjustments needed and adds our bootstrapping data-plane code to your Graphene image.
See [Building a service: Graphene]({{< ref "docs/building-services/graphene.md" >}}) for detailed information on Marblerun’s Graphene integration and our changes in your Graphene manifest.

Please note that this only works on a best-effort basis and may not instantly work correctly.
While suggestions should be made for every valid TOML Graphene configuration, changes can only be performed for non-hierarchically sorted configurations. as the official Graphene examples.
The unmodified manifest is saved as a backup under the old path with an added ".bak" suffix, allowing you to try out and roll back any changes performed.

Remember, you need to create a [Marblerun manifest]({{< ref "docs/workflows/define-manifest.md" >}}) in addition to the Graphene manifest. Adding Graphene packages to your manifest is straightforward and follows the same principles as any other SGX enclave. If you configured the arguments to your Graphene application through the [Graphene manifest](https://graphene.readthedocs.io/en/latest/manifest-syntax.html#command-line-arguments) before, you need to transfer those to the [Marblerun manifest]({{< ref "docs/workflows/define-manifest.md#manifestmarbles">}}).

  **Usage**

  ```bash
  marblerun graphene-prepare <path>
  ```

  **Examples**
  ```bash
  marblerun graphene-prepare nginx.manifest.template
  ```

  Output:
  ```bash
  Reading file: nginx.manifest.template

  Marblerun suggests the following changes to your Graphene manifest:
  libos.entrypoint = "file:premain-graphene"
  loader.argv0_override = "$(INSTALL_DIR)/sbin/nginx"
  loader.insecure__use_host_env = 1
  sgx.allowed_files.marblerun_uuid = "file:uuid"
  sgx.enclave_size = "1024M"
  sgx.remote_attestation = 1
  sgx.thread_num = 16
  sgx.trusted_files.marblerun_premain = "file:premain-graphene"
  Do you want to automatically apply the suggested changes [y/n]? y
  Applying changes...
  Saving original manifest as nginx.manifest.template.bak...
  Saving changes to nginx.manifest.template...
  Downloading Marblerun premain from GitHub...
  Successfully downloaded premain-graphene.

  Done! You should be good to go for Marblerun!
  ```

## Command `install`

Install Marblerun on your Kubernetes cluster.
This command will add Marblerun to your local helm repository if it is not present yet, optionally you can provide a path to your own helm chart.

**Usage**

```bash
marblerun install [flags]
```

**Flags**

{{<table "table table-striped table-bordered">}}
| Name, shorthand          | Default           | Description                                                                                                                                                  |
| :----------------------- | :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| --client-server-port     | 4433              | Set the client server port. Needs to be configured to the same <br> port as in your client tool stack                                                        |
| --disable-auto-injection |                   | Disable automatic injection of selected namespaces                                                                                                           |
| --domain                 | localhost         | Sets the CNAME for the coordinator certificate                                                                                                               |
| --help, -h               |                   | help for install                                                                                                                                             |
| --marblerun-chart-path   |                   | Path to marblerun helm chart                                                                                                                                 |
| --mesh-sever-port        | 2001              | Set the mesh server port. Needs to be configured to the same <br> port as in the data-plane marbles                                                          |
| --resource-key           | sgx.intel.com/epc | Resource providing SGX, different depending on used device plugin. Use this to set tolerations/resources if your device plugin is not supported by marblerun |
| --simulation             |                   | Set Marblerun to start in simulation mode, needed when not <br> running on an SGX enabled cluster                                                            |
| --version                |                   | Version of the Coordinator to install, latest by default                                                                                                     |
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
    Setting up Marblerun Webhook... Done
    Marblerun installed successfully
    ```

* Install Marblerun on a cluster without SGX Support (simulation mode)

    ```bash
    marblerun install --simulation
    ```

  The output is similar to the following:

  ```bash
  Setting up Marblerun Webhook... Done
  Marblerun installed successfully
  ```

## Command `manifest`

Set or update a manifest, or retrieve the signature of the manifest in place.

**Flags**
These flags apply to all subcommands of manifest

{{<table "table table-striped table-bordered">}}
| Name, shorthand | Default | Description                                                                                                                      |
| --------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| --era-config    |         | Path to remote attestation config file in json format, if none <br> provided the newest configuration will be loaded from github |
| --help, -h      |         | help for manifest                                                                                                                |
| --insecure, -i  |         | simulation mode                                                                                                                  |
{{</table>}}

* ### `set`

  Uploads a manifest in json or yaml format to the Marblerun coordinator.
  If a recovery key was set in the manifest, a recovery secret will be sent back.

  **Usage**

  ```bash
  marblerun manifest set <manifest.json> <IP:PORT> [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand     | Default | Description                                                |
  | ------------------- | ------- | ---------------------------------------------------------- |
  | --recovery-data, -r |         | File to write recovery data to, print to stdout if not set |
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

* ### `update`

  Update a manifest by uploading an update manifest to the Marblerun coordinator.
  The original manifest has to define one or multiple Users who are allowed to update the manifest.
  For more information see [Update]({{< ref "docs/workflows/update-manifest.md" >}})

  **Usage**

  ```bash
  marblerun manifest update <manifest.json> <IP:PORT> --cert=admin-cert.pem --key=admin-key.pem [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand | Default | Description                                   |
  | --------------- | ------- | --------------------------------------------- |
  | --cert, -c      |         | PEM encoded admin certificate file (required) |
  | --key, -k       |         | PEM encoded admin key file (required)         |
  {{</table>}}

  **Examples**

  ```bash
  marblerun manifest update update-manifest.json $MARBLERUN --cert=admin-cert.pem --key=admin-key.pem --era-config=era.json
  ```

  The output is the following:

  ```bash
  Successfully verified coordinator, now uploading manifest
  Manifest successfully updated
  ```

* ### `get`

  Retrieves the signature of an uploaded manifest. This allows a user to verify what manifest is running on the coordinator.

  **Usage**

  ```bash
  marblerun manifest get <IP:PORT> [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand | Default        | Description             |
  | --------------- | -------------- | ----------------------- |
  | --output, -o    | signature.json | Define file to write to |
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

* ### `signature`

  Print the signature of a Marblerun manifest.
  The manifest can be in either json or yaml format.

  **Usage**

  ```bash
  marblerun manifest signature manifest.json
  ```

  The output is the sha256 hash in base64 encoding of the manifest as it would be interpreted by the Marblerun coordinator.
  Note, that Internally, the coordinator handles the manifest in JSON format. Hence, the signature is always based on the JSON format of your manifest.
  You can quickly verify the integrity of the installed manifest by comparing the output of `marblerun manifest signature` on your local version and the signature returned by `marblerun manifest get` of the coordinator's version.

## Command `namespace`

Add namespaces to Marblerun.
If the auto-injection feature is enabled. All new pods in those namespaces will get their Marblerun configuration automatically injected.

* ### `add`

  Add a namespace to the Marblerun mesh by creating a new label

  **Usage**

  ```bash
  marblerun namespace add NAMESPACE ... [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand | Default | Description                                                             |
  | --------------- | ------- | ----------------------------------------------------------------------- |
  | --inject-sgx    |         | Set to enable automatic injection of SGX tolerations for <br> namespace |
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

* ### `remove`

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
  Namespace [default] successfully removed from the Marblerun mesh
  ```

* ### `list`

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

## Command `precheck`

  Check if your Kubernetes cluster supports SGX.
  More precisely the command will check if any nodes in the cluster define SGX resources through the use of [Device Plugins](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/).
  Currently supported are:
  * [Intel SGX Device Plugin](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/sgx_plugin/README.html), exposing the resources:
    * `sgx.intel.com/enclave`
    * `sgx.intel.com/epc`
    * `sgx.intel.com/provision`


  * [Azure SGX Device Plugin](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-aks-overview#azure-device-plugin-for-intel-sgx-), exposing the resource:
    * `kubernetes.azure.com/sgx_epc_mem_in_MiB`

  **Usage**

  ```bash
  marblerun precheck
  ```

  * If your cluster does not support SGX the output is the following:

  ```bash
  Cluster does not support SGX, you may still run Marblerun in simulation mode
  To install Marblerun run [marblerun install --simulation]
  ```

  * If your cluster does support SGX the output is similar to the following

  ```bash
  Cluster supports SGX on 2 nodes
  To install Marblerun run [marblerun install]
  ```

## Command `recover`

Recover the Marblerun coordinator from a sealed state by uploading a recovery key.
For more information about coordinator recovery see [Recovery]({{< ref "docs/workflows/recover-coordinator.md" >}})

**Usage**

```bash
marblerun recover <recovery_key_decrypted> <IP:PORT> [flags]
```

**Flags**

{{<table "table table-striped table-bordered">}}
| Name, shorthand | Default | Description                                                                                                                      |
| --------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| --era-config    |         | Path to remote attestation config file in json format, if none <br> provided the newest configuration will be loaded from github |
| --help, -h      |         | help for recover                                                                                                                 |
| --insecure, -i  |         | Set to skip quote verification, needed when running in <br> simulation mode                                                      |
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

## Command `secret`

Manages secrets for the Coordinator

**Flags**
These flags apply to all `secret` subcommands

{{<table "table table-striped table-bordered">}}
| Name, shorthand | Default | Description                                                                                                                 |
| --------------- | ------- | --------------------------------------------------------------------------------------------------------------------------- |
| --cert, -c      |         | PEM encoded Marblerun user certificate file (required)                                                                      |
| --era-config    |         | Path to remote attestation config file in json format, if none provided the newest configuration will be loaded from github |
| --insecure, -i  |         | Set to skip quote verification, needed when running in simulation mode                                                      |
| --key, -k       |         | PEM encoded Marblerun user key file (required)                                                                              |
{{</table>}}

* ### `get`

  Retrieves one or more secrets from the coordinator. Requires credentials in the form of a private key and self-signed certificate of the corresponding public key. The corresponding user needs to be permitted to access the requested secrets.
  Secrets are returned in JSON format with key data in base64 encoding.

  **Usage**

  ```bash
  marblerun secret get SECRETNAME ... <IP:PORT> [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand | Default | Description                |
  | --------------- | ------- | -------------------------- |
  | --output, -o    |         | File to save the secret to |
  {{</table>}}

  **Examples**

  ```bash
  marblerun secret get generic_secret symmetric_key_shared $MARBLERUN -c admin.crt -k admin.key
  ```

  The output is similar to the following:

  ```
  generic_secret:
  	Type:          plain
  	Data:          SGVsbG8gZnJvbSB0aGUgTWFyYmxlcnVuIERvY3MhCg==

  symmetric_key_shared:
  	Type:          symmetric-key
  	UserDefined:   false
  	Size:          128
  	Key:           uVGpoJZTRICLccJiVNt9jA==
  ```

* ### `set`

  Sets one or more secrets for the coordinator. Requires credentials in the form of a private key and a self-signed certificate of the corresponding public key. The corresponding user needs to be permitted to access the requested secrets.
  Secrets to set are specified in a special secrets file in JSON format, or created by the CLI from a PEM encoded certificate and key.
  For more information see [Managing Secrets]({{< ref "docs/workflows/managing-secrets.md" >}}).

  **Usage**

  ```bash
  marblerun secret set <secret.json> <IP:PORT> [flags]
  ```

  **Flags**

  {{<table "table table-striped table-bordered">}}
  | Name, shorthand | Default | Description                                  |
  | --------------- | ------- | -------------------------------------------- |
  | --from-pem      |         | set to load a secret from a PEM encoded file |
  {{</table>}}

  **Examples**

  ```bash
  marblerun secret set secret.json $MARBLERUN -c admin.crt -k admin.key
  ```

  ```bash
  marblerun secret set certificate_secret $MARBLERUN -c admin.crt -k admin.key --from-pem certificate.pem
  ```

  The output is the following:
  ```
  Secret successfully set
  ```

## Command `status`

Checks on the current status of the coordinator.

**Usage**

```bash
marblerun status <IP:PORT> [flags]
```

**Flags**

{{<table "table table-striped table-bordered">}}
| Name, shorthand | Default | Description                                                                                                                      |
| --------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| --era-config    |         | Path to remote attestation config file in json format, if none <br> provided the newest configuration will be loaded from github |
| --help, -h      |         | help for status                                                                                                                  |
| --insecure, -i  |         | Set to skip quote verification, needed when running in <br> simulation mode                                                      |
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

## Command `uninstall`

  Remove Marblerun from your Kubernetes cluster.
  This command will remove all resources added by the installation command.

  **Usage**

  ```bash
  marblerun uninstall
  ```

  The output is the following:
  ```bash
  Marblerun successfully removed from your cluster
  ```

## Command `version`

  Display version information of CLI, and the Marblerun coordinator running on a Kubernetes cluster.

  **Usage**

  ```bash
  marblerun version
  ```

  The output is similar to the following:

  ```
  CLI Version: v0.3.0
  Commit: 689787ea6f3ea3e047a68e2d4deaf095d1d84db9
  Coordinator Version: v0.3.0
  ```
