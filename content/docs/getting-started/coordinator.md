---
title: "Coordinator"
date: 2020-11-19T16:30:52+01:00
draft: false
weight: 5
---

# Coordinator

The Coordinator represents the control plane in Marblerun.
It communicates with the data plane through gRPC and provides an HTTP-REST interface on the client-side.
The Coordinator can be configured with several environment variables:

* `EDG_COORDINATOR_MESH_ADDR`: The listener address for the gRPC server
* `EDG_COORDINATOR_CLIENT_ADDR`: The listener address for the HTTP server
* `EDG_COORDINATOR_DNS_NAMES`: The DNS names for the cluster's root certificate
* `EDG_COORDINATOR_SEAL_DIR`: The file path for storing sealed data

## Client API

The Client API is designed as an HTTP-REST interface.
The API currently contains two endpoints:

* `/manifest`: For deploying and verifying the Manifest
    * Example for setting the Manifest:

    ```bash
    curl --cacert marblerun.crt --data-binary @manifest.json "https://$MARBLERUN/manifest"
    ```

    * Example for verifying the deployed Manifest

    ```bash
    curl --cacert marblerun.crt "https://$MARBLERUN/manifest" | jq '.ManifestSignature' --raw-output
    ```

* `/quote`: For retrieving a remote attestation quote over the whole cluster and the root certificate
    * Example for retrieving a quote

    ```bash
    curl -k "https://$MARBLERUN/quote"
    ```

    * We provide a tool to automatically verify the quote and output the trusted certificate:

    ```bash
    go install github.com/edgelesssys/era/cmd/era
    era -c coordinator-era.json -h $MARBLERUN -o marblerun.crt
    ```

        * Note that `coordinator-era.json` contains the *Packages* information for the Coordinator. For our testing image this can be pulled from our GitHub releases:

    ```bash
    wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
    ```
    