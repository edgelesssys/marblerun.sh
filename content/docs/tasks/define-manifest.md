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

## Manifest:Secrets
This section allows to define own secrets which are automatically generated upon the Coordinator's (if shared) or a Marble's (if not shared) launch and obtainable in the `Parameters` section similar to the placeholders values via the `.Secrets` prefix, following this convention:

`{{<output-type> .Secrets.<name>.<value>}}`

The following output options are available (replace with `<output-type>` in the example):
* `raw`: Returns the requested value as raw bytes.
* `hex`: Returns the requested value as a hex string.
* `base64`: Returns the requested value encoded in Base64.
* `pem`: Returns the requested value as a PEM (with the header matching the requested value).

And the following values can be queried from a secret (replace with `<value>` in the example):
* `<none>`: For secret type `raw`, returns the symmetric key. For other types, returns the public key.
* `Cert`: Returns the certificate (if available for given type)
* `Public`: Returns the public key (for secret type `raw`: returns the symmetric key)
* `Private`: Returns the private key (for secret type `raw`: returns the symmetric key)

Some examples based on the example secrets specified down below:

`{{ pem .Secrets.rsa_cert.Cert }}` (returns a PEM-encoded RSA certificate)

`{{ pem .Secrets.rsa_cert.Public }}` (returns the PKIX encoded public key of the generated certificate as PEM)

`{{ raw .Secrets.rsa_cert.Private }}` (returns the PKCS#8 encoded private key of the generated certificate as raw bytes)

`{{ hex .Secrets.secret_aes_key }}` (returns a randomly generated 16 byte symmetric key as a hex string)



The following parameters are available:
* `Type`: Can be either `raw` (symmetric key), `cert-rsa`, `cert-ecdsa` or `cert-ed25519`
* `Size`: Defines the size of the key used to generate the secret. As common for symmetric keys, for a secret of type `raw`, the size needs to be divisible by 8. For an ECDSA certificate, the size needs to map to a valid curve supported by Go's crypto library, which are currently (P-)224, 256, 384, or 521. For an ed25519 certificate, the parameter `size` needs to be omitted as in this case, the elliptic key used is always of the size 256.
* `Shared` (default: `false`): Specifies if the secret should be shared across all Marbles (`true`), or if the secret should be uniquely generated for each Marble (`false`). Keep in mind that secrets confined to a Marble come with certain limitations by now. For more information, look up the section [Secrets management]({{< ref "docs/features/secrets-management.md" >}}).
* `ValidFor` (only for certificates, default: `365`): Defines how long the certificate should be valid after generation in days. If not specified, a default value of `365` (365 days) is used. Please note that this field cannot be specified in combination with the `NotAfter` field in `Cert`. Only one of them can be defined.
* `Cert` (only for certificates): Allows the user to specify parameters for the x509 certificate which should be generated. This maps directly to a Go x509.Certificate object and every supported value can be specified, though certain ones (listed below) will either be filled automatically if left empty or even get replaced.
### Example of the Secrets section
```javascript
{
    //...
    "Secrets": {
        "secret_aes_key": {
            "Type": "raw",
            "Size": 128,
            "Shared": true
        },
        "rsa_cert": {
            "Type": "cert-rsa",
            "Size": 2048,
            "Shared": false,
            "ValidFor": 7,
            "Cert": {
                "SerialNumber": 42,
                "Subject": {
                    "SerialNumber": "42",
                    "CommonName": "Marblerun Unit Test"
                }
            }
        }
    }
    //...
}
```
### Struct of the `Cert` field (Go x509.Certificate)
```javascript
"Cert": {
        "SignatureAlgorithm": 0,
        "SerialNumber": null,
        "Subject": {
            "Country": null,
            "Organization": null,
            "OrganizationalUnit": null,
            "Locality": null,
            "Province": null,
            "StreetAddress": null,
            "PostalCode": null,
            "SerialNumber": "",
            "CommonName": "",
            "Names": null,
            "ExtraNames": null
        },
        "NotAfter": "0001-01-01T00:00:00Z",
        "KeyUsage": 0,
        "ExtKeyUsage": null,
        "UnknownExtKeyUsage": null,
        "MaxPathLen": 0,
        "MaxPathLenZero": false,
        "SubjectKeyId": null,
        "AuthorityKeyId": null,
        "OCSPServer": null,
        "IssuingCertificateURL": null,
        "DNSNames": null,
        "EmailAddresses": null,
        "IPAddresses": null,
        "URIs": null,
        "PermittedDNSDomainsCritical": false,
        "PermittedDNSDomains": null,
        "ExcludedDNSDomains": null,
        "PermittedIPRanges": null,
        "ExcludedIPRanges": null,
        "PermittedEmailAddresses": null,
        "ExcludedEmailAddresses": null,
        "PermittedURIDomains": null,
        "ExcludedURIDomains": null,
        "CRLDistributionPoints": null,
        "PolicyIdentifiers": null
    }
```
Of course, not every value needs to be defined. Some values you might want to fill out:
* `DNSNames`
* `IPAdresses`
* `KeyUsage` & `ExtKeyUsage`
* `Subject` (+ children)

The following values are always overwritten:
* `IsCA` (always `false`)
* `Issuer` (replaced with the Coordinator's Root CA, as it will issue the requested certificate)
* `BasicConstraintsValid` (always `true`)
* `NotBefore` (will be set to the current time during generation)


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
