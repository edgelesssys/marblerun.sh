---
title: "Verifying a deployment"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 8
---

# Verifying a deployment

An important feature of Marblerun is providing the ability to verifying the confidentiality and integrity of the whole application on the client-side.
To that end, we provide a simple REST-API that clients can use before interacting with the application.

## Establishing trust

The first step is to establish trust with the whole microservice mesh.
Therefore, Marblerun exposes the `/quote` endpoint that returns a quote and a certificate chain consisting of a root CA and an intermediate CA for the whole mesh. The root CA stays fixed over the whole lifetime of the Coordinator's configuration, while the intermediate CA changes in case you update the packages specified in your Manifest. For more information, see [Updating a Manifest]({{< ref "docs/tasks/update-manifest.md" >}}).

Verifying the quote can be done manually, but to ease the process we provide the Edgeless Remote Attestation tools ([era](https://github.com/edgelesssys/era)) for this purpose:

```bash
# Either install era for the current user
wget -P ~/.local/bin https://github.com/edgelesssys/era/releases/latest/download/era
chmod +x ~/.local/bin/era

# Or install it globally on your machine (requires root permissions)
sudo -O /usr/local/bin/era https://github.com/edgelesssys/era/releases/latest/download/era
sudo chmod +x /usr/local/bin/era

# Run era. You can remove the output parameters you do not need for your use case.
era -c coordinator-era.json -h $MARBLERUN -output-chain marblerun-chain.pem -output-root marblerun-root.pem -output-intermediate marblerun-intermedite.pem
```

*Note: On machines running Ubuntu, ~/.local/bin is only added to PATH when the directory exists when initializing your bash environment during login. You might need to re-login after creating the directory. Also, non-default shells such as `zsh` do not add this path by default. Therefore, if you receive `command not found: era` as an error message for a local user installation, either make sure ~/.local/bin was added to your PATH successfully or simply use the machine-wide installation method.*

era requires the Coordinator's UniqueID and SignerID (or MRENCLAVE and MRSIGNER in SGX terms) to verify the quote.
In production, these would be generated when building Coordinator and distributed to your clients.
For testing, we have published a Coordinator image at `ghcr.io/edgelesssys/coordinator`.
You can pull the corresponding `coordinator-era.json` file from our release page:

```bash
wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
```

After successful verification, you'll have `marblerun-chain.pem`, `marblerun-root.pem` and `marblerun-intermediate.pem` in your directory which you can choose to use for your application, depending on use case. In case you want to pin against specific versions of your application, using the intermediate CA as a trust anchor is a good choice. If this is not a critical issue for you, you can pin against the root CA in which case different versions of your application can talk with each other, though you may not be able to launch them if they do not meet the minimum `SecurityVersion` specified in your original or updated Manifest.

## Verifying the Manifest

Establishing trust with the service mesh allows you to verify the deployed Manifest in the second step.
To that end, Marblerun exposes the endpoint `/manifest`.
Using the CLI you can get the Manifest's signature aka its sha256 hash:

```bash
marblerun manifest get $MARBLERUN -o manifest-signature.json
cat manifest-signature.json | jq '.data.ManifestSignature' --raw-output
```

Compare this against your local version of the Manifest:

```bash
sha256sum manifest.json
```
