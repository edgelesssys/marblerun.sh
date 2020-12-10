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
# If you have EdgelessRT installed
go install github.com/edgelesssys/era/cmd/era
# Or use the binary release
wget -O .local/bin/era https://github.com/edgelesssys/era/releases/latest/download/era
# Run era
era -c coordinator-era.json -h $MARBLERUN -o marblerun.crt
```

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