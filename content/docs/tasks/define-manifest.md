---
title: "Defining a Manifest"
date: 2020-11-19T16:44:36+01:00
draft: false
weight: 2
---

# Defining a Manifest

The Manifest is a simple JSON file that determines the key properties of your cluster: `Packages`, `Infrastructures`, `Marbles`, and `RecoveryKey`.
This article describes how to define these in your `manifest.json`.

## Manifest:Packages

A package defines a specific container image in your application.
It contains the secure enclave's measurements and associated properties:

* **UniqueID**: The enclave's unique identifying measurement, called MRENCLAVE on SGX
* **SignerID**: The signer's unique identifier, called MRSIGNER on SGX
* **ProductID**: The unique identifier of your product associated with the enclave
* **SecurityVersion**: The version number of your product associated with the enclave
* **Debug**: A flag indicating whether your enclave should be run in debug mode

You can use any combination of these values depending on how you want to identify the image.
For each confidential container you want to run in your cluster, you need to add an entry in the *Packages* section of the Manifest.

```javascript
{
    // ...
    "Packages": {
        "backend": {
            "UniqueID": "000102030405060708090a0b0c0d0e0f101112131415161718191a1b1c1d1e1f",
            "ProductID": 42,
            "SecurityVersion": 1,
            "Debug": false
        },
        "frontend": {
            "SignerID": "c0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffeec0ffee",
            "ProductID": 43,
            "SecurityVersion": 3,
            "Debug": true
        }
    }
    // ...
}
```

## Manifest:Marbles

Marbles represent the actual services in your mesh. They are defined in the *Marbles* section. Each Marble corresponds to a `Package` and defines a set of optional `Parameters`:

* Files: Files and their contents
* Env: Environment variables
* Argv: Command line arguments

These `Parameters` are passed from the Coordinator to secure enclaves after successful initial remote attestation. `Parameters` can contain the following placeholders:

* `.Marblerun.RootCA.Cert`: The root certificate of the cluster issued by the Coordinator; it can be used to verify the certificates of all Marbles in the cluster.
* `.Marblerun.MarbleCert.Cert`: The Marble's certificate; issued by the Coordinator and used for Marble-to-Marble and Marble-to-client authentication
* `.Marblerun.MarbleCert.Private`: The private key corresponding to `MarbleCert`
* `.Marblerun.SealKey`: A 128-bit symmetric encryption key that can be used for sealing data to disk in a host-independent way; if a Marble is scheduled or restarted on a new host, this "virtual sealing key" will still allow for unsealing data from the disk even though the host's actual sealing key might have changed.

```javascript
{
    // ...
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
    }
    //...
}
```

## Manifest:RecoveryKey

The optional entry `RecoveryKey` holds a X.509 PEM-encoded RSA public key, which can be used to recover a failed Marblerun deployment, as is described [here]({{< ref "docs/features/recovery.md" >}}).

```javascript
{
    //...
    "RecoveryKey": "-----BEGIN PUBLIC KEY-----\nMIIBpTANBgk..."
}
```

This key can be generated with the help of OpenSSL.

```bash
openssl genrsa -out private_key.pem 4096
openssl rsa -in private_key.pem -outform PEM -pubout -out public_key.pem
```

To preserve the new lines correctly, you can use the following command:

```bash
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' public_key.pem
```

## Manifest:Infrastructures

Future versions of Marblerun will allow you to define certain trusted infrastructures and hardware configurations under `Infrastructures`.

