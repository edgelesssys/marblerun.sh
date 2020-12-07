---
title: "Setting a Manifest"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 3
---

# Setting a Manifest

The Manifest is a JSON document that defines which services span the mesh and how they should be configured.
It further defines what Infrastructure providers are allowed.
You can set a Manifest through Marblerun's Client REST-API.
The endpoint for all Manifest operations is `/manifest`.

See the following Manifest for example (`manifest.json`).

```json
{
    "Packages": {
        "backend": {
            "UniqueID": "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f",
            "ProductID": 43,
            "SecurityVersion": 1,
            "Debug": false
        },
        "frontend": {
            "SignerID": "c0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffee",
            "ProductID": 42,
            "SecurityVersion": 3,
            "Debug": true
        }
    },
    "Marbles": {
        "backend_first": {
            "Package": "backend",
            "MaxActivations": 1,
            "Parameters": {
                "Files": {
                    "/tmp/defg.txt": "foo",
                    "/tmp/jkl.mno": "bar"
                },
                "Env": {
                    "IS_FIRST": "true",
                    "ROOT_CA": "{{ pem .Marblerun.RootCA.Cert }}",
                    "SEAL_KEY": "{{ hex .Marblerun.SealKey }}",
                    "MARBLE_CERT": "{{ pem .Marblerun.MarbleCert.Cert }}",
                    "MARBLE_KEY": "{{ pem .Marblerun.MarbleCert.Private }}"
                },
                "Argv": [
                    "--first",
                    "serve"
                ]
            }
        },
        "backend_other": {
            "Package": "backend",
            "Parameters": {
                "Env": {
                    "ROOT_CA": "{{ pem .Marblerun.RootCA.Cert }}",
                    "SEAL_KEY": "{{ hex .Marblerun.SealKey }}",
                    "MARBLE_CERT": "{{ pem .Marblerun.MarbleCert.Cert }}",
                    "MARBLE_KEY": "{{ pem .Marblerun.MarbleCert.Private }}"
                },
                "Argv": [
                    "serve"
                ]
            }
        },
        "frontend": {
            "Package": "frontend",
            "Parameters": {
                "Env": {
                    "ROOT_CA": "{{ pem .Marblerun.RootCA.Cert }}",
                    "SEAL_KEY": "{{ hex .Marblerun.SealKey }}",
                    "MARBLE_CERT": "{{ pem .Marblerun.MarbleCert.Cert }}",
                    "MARBLE_KEY": "{{ pem .Marblerun.MarbleCert.Private }}"
                }
            }
        }
    },
    "RecoveryKey": "-----BEGIN PUBLIC KEY-----\nMIIBpTANBgkqhkiG9w0BAQEFAAOCAZIAMIIBjQKCAYQAyokHE545y3lU4xsxrqXJ\n58jiaXN8yEdjjuKk0903zMT+FV62UeX17BQhrtdOIf4l4/V/xipqI+osAHBQpRY1\nwM1NCIFFlXUQGgXdtoWiAS7zfFKC+mNlB63Z0Z/50Iw9pl6AFWBQ+16lfmsPMnIu\nLHf4AL3KXVlpgPn6cmRfUoDBx6ITm2QrCDFlVu4j4isgnaZrw6VD0V+G9Mcpgs/0\n0XNmz72eMULfuW+4ULJI9Fx88wiNWWHeSI4vz83ylM5+1QntFROSYWBjgmCnm25j\nKbzV765CVTIU3qq3qkYmclpHfKKt7/TOgVOauvkMCYXyLJkSd1LGLIctWK8tCs1K\nnB237nNg+dZ67Zz9lBYKfNnFoudoc85+vXBRKIfV56FXiXrB32hF1DEj11viMPUr\nroMokLFtDCoAk0Xok4AFQDOgxTw7F8cHskjIYWVCmCqmDUI+FGttyVrc5YLSHAuR\nxQ2oxD0F44JXwxDc/C+OYzOApYl25rmR2nuqioDGpL6/ELRRAgMBAAE=\n-----END PUBLIC KEY-----\n"
}
```

For setting the Manifest, we first need to establish trust in the Coordinator.
Therefore, we perform a remote attestation step.
Assuming you've deployed our Coordinator image from `ghcr.io/edgelesssys/coordinator`:

1. Pull the UniqueID and SignerID values for this image:

    ```bash
    wget https://github.com/edgelesssys/marblerun/releases/latest/download/coordinator-era.json
    ```

2. Use the Edgeless Remote Attestation tool to verify the Mesh's quote and get a trusted certificate:

    ```bash
    era -c coordinator-era.json -h $MARBLERUN -o marblerun.crt
    ```

3. Now that we have established trust, we can set the Manifest through the Client API:

    ```bash
    curl --cacert marblerun.crt --data-binary @manifest.json "https://$MARBLERUN/manifest"
    ```

If the Manifest contains a `RecoveryKey` entry, you will receive a JSON reply including a recovery secret, encrypted with for the `RecoveryKey`. The reply will look like this, with `[base64]` as your encrypted recovery secret.

`{"EncryptionKey":"[base64]"}`

**It is important that you keep this value stored somewhere safe. Without it, you will not be able to perform a recovery step in case the SGX seal key changed.**