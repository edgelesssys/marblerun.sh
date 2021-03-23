---
title: "Coordinator"
date: 2020-11-19T16:30:52+01:00
draft: false
weight: 3
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

The Coordinator clients can be devided into two major groups.

* The owners/providers/administrators who need to interact with the coordinator for deploying their confidential application and administrative tasks
* The users/customers who use the coordinator for remote attestation and establishing trust with the application

The Client API serves both use-cases with a compact REST-API.

### Response style

The Client API is designed as an HTTP-REST interface.Responses follow the [JSend](https://github.com/omniti-labs/jsend) style, though only the response types `success` and `error` are returned so far.

In general, a successful API call (HTTP Code 200) will return a response in the following style:

```json
{
    "status": "success",
    "data": {
        "ManifestSignature": "3fff78e99dd9bd801e0a3a22b7f7a24a492302c4d00546d18c7f7ed6e26e95c3"
    }
}
```

Depending on the API endpoint and the data submitted, `data` might contain a specific answer from the coordinator, or may just be `null` to acknowledge that the requested operation was performed successfully.

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

The API currently contains the following endpoints. If an endpoint specifies *Returns* for either HTTP GET or HTTP POST, it means that the specified data can be found encoded inside the `data` block if the response was successful. If no returns are specified for a given endpoint, or in case all possible return values for an endpoint are declared as optional, `data` can just be `null`.

#### /manifest

For deploying and verifying the Manifest.

* Before deploying the application to the cluster the manifest needs to be set once by the provider
* Users can retrieve and inspect the manifest through this endpoint before interacting with the application

**Returns (HTTP GET)**:


{{<table "table table-striped table-bordered">}}
| Field value       | Type   | Description                                                                                        |
| ----------------- | ------ | -------------------------------------------------------------------------------------------------- |
| ManifestSignature | string | A SHA-256 of the currently set manifest. Does not change when an Update Manifest has been applied. |
{{</table>}}

**Returns (HTTP POST)**:

{{<table "table table-striped table-bordered">}}
| Field value     | Type             | Description                                                                                                                                                                                                |
| --------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| RecoverySecrets | array (optional) | An array containing key-value mapping for encrypted secrets to be used for recovering the Coordinator in case of disaster recovery. The key matches each supplied key from `RecoveryKeys` in the Manifest. |
{{</table>}}

Example for setting the Manifest (HTTP POST):

```bash
curl --cacert marblerun.crt --data-binary @manifest.json "https://$MARBLERUN/manifest"
```

Example for verifying the deployed Manifest (HTTP GET):

```bash
curl --cacert marblerun.crt "https://$MARBLERUN/manifest" | jq '.data.ManifestSignature' --raw-output
```

#### /quote

For retrieving a remote attestation quote over the whole cluster and the root certificate.
The quote is an SGX-DCAP quote, you can learn more about DCAP in the [official Intel DCAP orientation](https://download.01.org/intel-sgx/sgx-dcap/1.9/linux/docs/Intel_SGX_DCAP_ECDSA_Orientation.pdf).
Both the provider and the users of the confidential application can use this endpoint to verify the integrity of the Coordinator and the cluster at any time.

**Returns (HTTP GET)**:
{{<table "table table-striped table-bordered">}}
| Field value | Type   | Description                                                                                                                                                               |
| ----------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Cert        | string | A PEM-encoded certificate chain containing the Coordinator's Root CA and Intermediate CA, which can be used for trust establishment between a client and the Coordinator. |
| Quote       | string | Base64-encoded quote which can be used for Remote Attestation, as described in [Verifying a deployment]({{< ref "docs/tasks/verification.md" >}})                         |
{{</table>}}

Example for retrieving a quote

```bash
curl -k "https://$MARBLERUN/quote"
```

We provide a tool to automatically verify the quote and output the trusted certificate:

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

Note that `coordinator-era.json` contains the *Packages* information for the Coordinator. For our testing image this can be pulled from our GitHub releases:

```bash
wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
```

#### /recover

For recovering the Coordinator in case unsealing the existing state failed.

This API endpoint is only available when the coordinator is in recovery mode. Before you can use the endpoint, you need to decrypt the recovery secret which you may have received when setting the manifest initially. See [Recovering the Coordinator]({{< ref "docs/tasks/recover-coordinator.md" >}}) to retrieve the recovery key needed to use this API endpoint correctly.

Example for recovering the coordinator:

```bash
curl -k -X POST --data-binary @recovery_key_decrypted "https://$MARBLERUN/recover"
```

#### /status

For returning the current state of the coordinator.

**Returns (HTTP GET)**:
{{<table "table table-striped table-bordered">}}
| Field value   | Type   | Description                                                                                       |
| ------------- | ------ | ------------------------------------------------------------------------------------------------- |
| StatusCode    | int    | A status code which matches the internal code of the Coordinator's current state.                 |
| StatusMessage | string | A descriptive status message of what the Coordinator expects the user to do in its current state. |
{{</table>}}

**Possible values**:

{{<table "table table-striped table-bordered">}}
| StatusCode | StatusMessage                                                                                                                                                             |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1          | Coordinator is in recovery mode. Either upload a key to unseal the saved state, or set a new manifest. For more information on how to proceed, consult the documentation. |
| 2          | Coordinator is ready to accept a manifest.                                                                                                                                |
| 3          | Coordinator is running correctly and ready to accept marbles.                                                                                                             |
{{</table>}}

Example for getting the status:

```bash
curl -k "https://$MARBLERUN/status"
```

It may be useful to use this API endpoint and use it for other monitoring tools. More information can be found under [Monitoring and Logging]({{< ref "docs/tasks/monitoring.md" >}})

#### /update

For updating the packages specified in the currently set Manifest.

This API endpoint only works when `Admins` were defined in the Manifest. For more information, look up [Updating a Manifest]({{< ref "docs/tasks/update-manifest.md" >}})

Example for updating the manifest:

```bash
curl --cacert marblerun.crt --cert admin_certificate.crt --key admin_private.key -w "%{http_code}" --data-binary @update_manifest.json https://$MARBLERUN/update
```