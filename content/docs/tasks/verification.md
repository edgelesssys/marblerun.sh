---
title: "Verifying a deployment"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 5
---

# Verifying a deployment

An important feature of Marblerun is providing the ability to verifying the confidentiality and integrity of the whole application on the client-side.
To that end, we provide a simple REST-API that clients can use before interacting with the application.

## Establishing trust

The first step is to establish trust with the whole microservice mesh.
Therefore, Marblerun exposes the `/quote` endpoint that returns a quote and a root certificate for the whole mesh.
Verifying the quote can be done manually, but to ease the process we provide the Edgeless Remote Attestation tools ([era](https://github.com/edgelesssys/era)) for this purpose:

```bash
# Either install era for the current user
wget -P ~/.local/bin https://github.com/edgelesssys/era/releases/latest/download/era
chmod +x ~/.local/bin/era

# Or install it globally on your machine (requires root permissions)
sudo -O /usr/local/bin/era https://github.com/edgelesssys/era/releases/latest/download/era
sudo chmod +x /usr/local/bin/era

# Run era
era -c coordinator-era.json -h $MARBLERUN -o marblerun.crt
```

*Note: On machines running Ubuntu, ~/.local/bin is only added to PATH when the directory exists when initializing your bash environment during login. You might need to re-login after creating the directory. Also, non-default shells such as `zsh` do not add this path by default. Therefore, if you receive `command not found: era` as an error message for a local user installation, either make sure ~/.local/bin was added to your PATH successfully or simply use the machine-wide installation method.*

era requires the Coordinator's UniqueID and SignerID (or MRENCLAVE and MRSIGNER in SGX terms) to verify the quote.
In production, these would be generated when building Coordinator and distributed to your clients.
For testing, we have published a Coordinator image at `ghcr.io/edgelesssys/coordinator`.
You can pull the corresponding `coordinator-era.json` file from our release page:

```bash
    wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
```

After successful verification, you'll have the trusted root certificate `marblerun.crt` to use with your application.

## Verifing the Manifest

Establishing trust with the service mesh allows you to verify the deployed Manifest in the second step.
To that end, Marblerun exposes the endpoint `/manifest`.
Using curl you can get the Manifest's signature aka its sha256 hash:

```bash
curl --cacert marblerun.crt "https://$MARBLERUN/manifest" | jq '.ManifestSignature' --raw-output
```

Compare this against your local version of the Manifest:

```bash
sha256sum manifest.json
```
