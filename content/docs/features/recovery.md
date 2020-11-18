---
title: "Recovery"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 2
---

# Recovery

Persistent storage for confidential applications in the cloud requires a bit of attention.
Unfortunately, SGX sealing keys are unique to a single CPU, which means using the default SGX sealing methods has some caveats.
For example, sealing data while running on one node could mean the data can't be unsealed when running on another node later on.
The Coordinator provides Marbles with virtual sealing keys making persistence straightforward for your applications.
By using the virtual sealing key data can be unsealed independently of the nodes the app is running on.

Unfortunately, the Coordinator itself must keep its state persistent somehow. When being pinned to a single node the default SGX sealing methods are used. However, when the Coordinator needs to be shifted to  another node or another cluster a manual step is required to ensure the Coordinator's state can be recovered.
Therefore, the manifest allows specifying a recovery key.

## Key Generation

Marblerun allows specifying a PKIX PEM-encoded RSA Public Key in the manifest. Such key can be generated with the help of OpenSSL:

```bash
openssl genrsa -out private_key.pem 4096
openssl rsa -in private_key.pem -outform PEM -pubout -out public_key.pem
```

In the manifest, the PEM is stored as `RecoveryKey`. See [Setting a Manifest](tasks/set-manifest.md) as an example. To preserve the new lines correctly, you can use the following command:

```bash
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' public_key.pem
```

When setting a manifest through the `/manifest` API endpoint, you will receive a JSON reply including a recovery secret, encrypted with your RSA public key. The reply will look like this, with `[base64]` as your encrypted recovery secret.

`{"EncryptionKey":"[base64]"}`

**It is important that you keep this value stored somewhere safe. Without it, you will not be able to perform a recovery step in case the SGX seal key changed.**

## Restore State

If the Coordinator finds a sealed state during its startup which it is unable to unseal using the current SGX seal key it will wait for further instructions.
You have to options on how to proceed:

1. Recover the sealed state by uploading the recovery secret

    The recovery secret can be uploaded through the `/recover` client API endpoint. In order to do so a client needs to first extract the encrypted secret by decrypting it with the corresponding private key:

    ```bash
    base64 -d recovery_key_encrypted_base64 > recovery_key_encrypted
    openssl pkeyutl -inkey private_key.pem -in recovery_key_encrypted -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256 -decrypt -out recovery_key_decrypted
    ```

    The extracted secret can then be uploaded via the client API.

    ```bash
    curl -k -X POST --data-binary @recovery_key_decrypted "https://$MARBLERUN/recover"
    ```

    If the recovery worked correctly, the Coordinator should apply the sealed state again without returning an error. In case the Coordinator was not able to restore the state with the uploaded key, an error will be returned in the logs and the `/recover` endpoint will stay open for further interaction.

1. Dismiss the sealed state by uploading a new manifest

    In case there is no desire to recover the old state it can simply be dismissed by [uploading a new manifest](tasks/set-manifest.md).
    *Note* that if a new manifest is uploaded to the server, the old state will be *overwritten* on disk and the `/recover` endpoint will not be available anymore.