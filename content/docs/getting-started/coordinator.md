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

### Response Style
The Client API is designed as an HTTP-REST interface. Responses follow the [JSend](https://github.com/omniti-labs/jsend) style, though only the response types `success` and `error` are returned so far.

In general, a successful API call (HTTP Code 200) will return a response in the following style:
```json
{
    "status": "success",
    "data": {
        "ManifestSignature": "3fff78e99dd9bd801e0a3a22b7f7a24a492302c4d00546d18c7f7ed6e26e95c3"
    }
}
```
Depending on the API endpoint and the data submitted, `data` might contain a specific answer from the coordinator, or may just be `null` to acknowledge that the requested operation was performormed successfully.

Whereas an error (HTTP Code 4xx or 5xx) might look like this:
```json
{
    "status": "error",
    "data": null,
    "message": "server is not in expected state"
}
```
For errors, `data` will always be `null`, and `message` contains the specific error the Coordinator ran into when processing the request.

### Endpoints
The API currently contains the following endpoints:

* `/manifest`: For deploying and verifying the Manifest
    * Example for setting the Manifest (HTTP POST):

        ```bash
        curl --cacert marblerun.crt --data-binary @manifest.json "https://$MARBLERUN/manifest"
        ```

    * Example for verifying the deployed Manifest (HTTP GET):

        ```bash
        curl --cacert marblerun.crt "https://$MARBLERUN/manifest" | jq '.data.ManifestSignature' --raw-output
        ```

* `/quote`: For retrieving a remote attestation quote over the whole cluster and the root certificate
    * Example for retrieving a quote

        ```bash
        curl -k "https://$MARBLERUN/quote"
        ```

    * We provide a tool to automatically verify the quote and output the trusted certificate:

        ```bash
        # Either install era for the current user
        wget -P ~/.local/bin https://github.com/edgelesssys/era/releases/latest/download/era
        chmod +x ~/.local/bin/era

        # Or install it globally on your machine (requires root permissions)
        sudo -O /usr/local/bin/era https://github.com/edgelesssys/era/releases/latest/download/era
        sudo chmod +x /usr/local/bin/era

        era -c coordinator-era.json -h $MARBLERUN -o marblerun.crt
        ```

        *Note: On machines running Ubuntu, ~/.local/bin is only added to PATH when the directory exists when initializing your bash environment during login. You might need to re-login after creating the directory. Also, non-default shells such as `zsh` do not add this path by default. Therefore, if you receive `command not found: era` as an error message for a local user installation, either make sure ~/.local/bin was added to your PATH successfully or simply use the machine-wide installation method.*


    * Note that `coordinator-era.json` contains the *Packages* information for the Coordinator. For our testing image this can be pulled from our GitHub releases:

        ```bash
        wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
        ```

* `/recover`: For recovering the Coordinator in case unsealing the existing state failed.

    * This API endpoint is only available when the coordinator is in recovery mode.

    * Before you can use the endpoint, you need to decrypt the recovery secret which you may have received when setting the manifest initially. See [Recovering the Coordinator]({{< ref "docs/tasks/recover-coordinator.md" >}}) to retrieve the recovery key needed to use this API endpoint correctly.

    * Example for recovering the coordinator:
        ```bash
        curl -k -X POST --data-binary @recovery_key_decrypted "https://$MARBLERUN/recover"
        ```

* `/status`: For returning the current state of the coordinator.
    * Example for getting the status:
    ```bash
    curl -k "https://$MARBLERUN/status"
    ```

    * It may be useful to use this API endpoint and use it for other monitoring tools. More information can be found under [Monitoring and Logging]({{< ref "docs/tasks/monitoring.md" >}})

    * Possible status codes:

        | Code | Status                                                                                                                                                                                                                |
        |------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
        | 1    | Recovery Mode: The coordinator was unable to unseal an existing state and needs to be reset or recovered. Consult [Recovering the Coordinator]({{< ref "docs/tasks/recover-coordinator.md" >}}) for more information. |
        | 2    | Ready to accept a manifest over the /manifest endpoint.                                                                                                                                                               |
        | 3    | The coordinator is setup correctly and ready to launch marbles.                                                                                                                                                       |
        | -1   | An unknown error occured.                                                                                                                                                                                             |

* `/update`: For updating the packages specified in the currently set Manifest.

    * This API endpoint only works when `Admins` were defined in the Manifest. For more information, look up [Updating a Manifest]({{< ref "docs/tasks/update-manifest.md" >}})

    * Example for updating the manifest:

        ```bash
        curl --cacert marblerun.crt --cert admin_certificate.crt --key admin_private.key -w "%{http_code}" --data-binary @update_manifest.json https://$MARBLERUN/update
        ```
