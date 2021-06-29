---
title: "Defining a Manifest"
date: 2020-11-19T16:44:36+01:00
draft: false
weight: 1
---

# Defining a Manifest

The Manifest is a simple JSON file that determines the key properties of your cluster: `Packages`, `Marbles`, `Secrets`, and `RecoveryKeys`.
This article describes how to define these in your `manifest.json`.

## Manifest:Packages

The `Packages` section of the Manifest lists all the secure enclave software packages that your application uses. A package is defined by the following properties.

* `UniqueID`: this value will pin this package to one specific release build of an application. It represents the globally unique ID of the enclave software package; on SGX, this corresponds to the `MRENCLAVE` value, which is the SHA-256 hash of the enclave's initial contents and its configuration.
* `SignerID`: this value limits Marblerun to only accept releases signed by a given public key. On SGX, this corresponds to the `MRSIGNER` value, which is the SHA-256 hash of the enclave issuer's RSA-3072 public key.
* `ProductID`: an integer that uniquely identifies the enclave software for a given `SignerID`. Can only be used in conjunction with `SignerID`.
* `SecurityVersion`: an integer that reflects the security-patch level of the enclave software. Can only be used in conjunction with `SignerID`.
* `Debug`: set to `true` if the enclave is to be run in debug mode. This allows you to experiment with deploying your application with Marblerun without having to worry about setting correct values for the above properties, but note that enclaves in debug mode are not secure.

The following gives an example of a simple `Packages` section with made-up values.

```javascript
{
    // ...
    "Packages": {
        "pkg0": {
            "UniqueID": "6b2822ac2585040d4b9397675d54977a71ef292ab5b3c0a6acceca26074ae585",
            "Debug": false
        },
        "pkg1": {
            "SignerID": "43361affedeb75affee9baec7e054a5e14883213e5a121b67d74a0e12e9d2b7a",
            "ProductID": 43,
            "SecurityVersion": 3,
            "Debug": true
        }
    }
    // ...
}
```

In this example, `pkg0` is identified through `UniqueID`. Since `UniqueID` is the hash of the enclave software package, this means that `pkg0` cannot be updated. (That is because any update to the package will change the hash.)

In contrast, `pkg1` is identified through the triplet `SignerID`, `ProductID`, and `SecurityVersion`. `SignerID` cryptographically identifies the vendor of the package; `ProductID` is an arbitrary product ID chosen by the vendor, and `SecurityVersion` is the security-patch level of the product. See our [adding a service hands-on]({{< ref "docs/workflows/add-service.md#step-21-define-the-enclave-software package" >}}) on how to get these values for a given service.

Future versions of Marblerun will accept any `SecurityVersion` that is equal or higher than the one specified in `Packages` for a given combination of `SignerID` and `ProductID`. This way, updates to packages can be made without having to alter the Manifest.

## Manifest:Marbles

Marbles represent the actual services in your mesh. They are defined in the `Marbles` section, which typically looks somewhat like the following example.

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
            },
            "TLS": [
                "backend_first_tls"
            ]
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
            },
            "TLS": [
                "frontend_tls_1", "frontend_tls_2"
            ]
        }
    }
    //...
}
```

Each Marble corresponds to a `Package` (see the [previous section](#manifestpackages)) and defines a set of optional `Parameters`:

* `Files`: Files and their contents
* `Env`: Environment variables
* `Argv`: Command line arguments
* `TLS`: Tags defined in the [`TLS` section](#manifesttls)

These `Parameters` are passed from the Coordinator to secure enclaves (i.e., Marbles) after successful initial remote attestation. In the remote attestation step, the Coordinator ensures that enclaves run the software-defined in the `Packages` section. It is important to note that `Parameters` are only accessible from within the corresponding secure enclave. `Parameters` may contain arbitrary static data. However, they can also be used to securely communicate different types of dynamically generated cryptographic keys and certificates to Marbles. For this, we use [Go Templates](https://golang.org/pkg/text/template/) with the following syntax.

`{{ <encoding> <name of secret> }}`

The following enconding types are available.

* `raw`: raw bytes
* `hex`: hex string
* `base64`: [Base64](https://de.wikipedia.org/wiki/Base64) encoding
* `pem`: [PEM](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) encoding with a header matching the type of the requested key or certificate

The following named keys and certificates are always available.

* `.Marblerun.RootCA.Cert`: the root certificate of the cluster issued by the Coordinator; this can be used to verify the certificates of all Marbles in the cluster.
* `.Marblerun.MarbleCert.Cert`: the Marble's certificate; this is issued by the `.Marblerun.RootCA.Cert` and is for Marble-to-Marble and Marble-to-client authentication.
* `.Marblerun.MarbleCert.Private`: the Marble's private key corresponding to `.Marblerun.MarbleCert.Cert`
* `.Marblerun.SealKey`: a 128-bit symmetric encryption key, which can be used for sealing data to disk in a host-independent way; if a Marble is scheduled or restarted on a new host, this "virtual sealing key" will still allow for unsealing data from the disk even though the host's actual sealing key might have changed.

Finally, the optional field `MaxActivations` can be used to restrict the number of distinct instances that can be created of a Marble.

## Manifest:Secrets

In the [previous section](#manifestmarbles), we discussed how certain cryptographic keys and certificates can be injected into a Marble's `Parameters` using Go Templates. In addition, Marblerun also allows for the specification of custom cryptographic keys and certificates in the `Secrets` section. A typical `Secrets` section looks like the following.

```javascript
{
    //...
    "Secrets": {
        "secret_aes_key": {
            "Type": "symmetric-key",
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
        },
        "secret_key_unset": {
            "Type": "symmetric-key",
            "UserDefined": true
        }
    }
    //...
}
```

When defining a custom key or certificate, the following fields are available.

* `Type`: can be either `symmetric-key` for a symmetric encryption key, `cert-rsa`, `cert-ecdsa`, `cert-ed25519` or `plain`. Secrets of type `plain` contain arbitrary data uploaded by users, and are never genrated by the coordinator.
* `Size`: the size of the key in bits. For symmetric keys, this needs to be a multiple of `8`. For ECDSA, this needs to map to a curve supported by Go's crypto library, currently: `224`, `256`, `384`, or `521`. For Ed25519, this should be ommitted.
* `Shared` (default: `false`): specifies if the secret should be shared across all Marbles (`true`), or if the secret should be uniquely generated for each Marble (`false`). See [Secrets management]({{< ref "docs/features/secrets-management.md" >}}) for more info.
* `ValidFor` (only for certificates, default: `365`): validity of the certificate in days; cannot be specified in combination with the `NotAfter`.
* `Cert` (only for certificates): allows for the specification of additional X.509 certificate properties. See below for details.
* `UserDefined` (default: `false`): specifies if the secret should be generated by Marblerun  (`false`), or if it will be uploaded by a user at a later date  (`true`).

### Available `Cert` fields

When specifying a custom certificate in the `Secrets` section, the following properties can be set. These map directly to Go's  `x509.Certificate` structure. (This is because the Coordinator is written in Go.)

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
Typically, you only define a subset of these. Commonly used properties include for example:
* `DNSNames`
* `IPAdresses`
* `KeyUsage` & `ExtKeyUsage`
* `Subject` (+ children)

The following X.509 properties cannot be specified because they are set by the Coordinator when creating a certificate.
* `IsCA`: always set to "false"
* `Issuer`: always set to "Marblerun Coordinator"
* `BasicConstraintsValid`: always set to "true"
* `NotBefore`: always set to the host time at creation

### Injecting custom secrets

Keys and certificates defined in the `Secrets` section can be injected via `Parameters` using the following syntax.

`{{ <encoding> .Secrets.<name>.<part> }}`

Refer to the [previous section](#manifestmarbles) for a list of supported encodings. `<part>` can be any of the following.

* *empty*: for secret type `symmetric-key`, returns the symmetric key. For secret type `plain`, returns the secret data. For other types, returns the public key.
* `Cert`: returns the certificate.
* `Public`: returns the public key.
* `Private`: returns the private key.

The following gives some examples.

* Inject the certificate of custom secret `rsa_cert` in PEM format: `{{ pem .Secrets.rsa_cert.Cert }}`
* Inject the corresponding private key in PKCS#8 format: `{{ raw .Secrets.rsa_cert.Private }}`
* Inject the corresponding public key PKIX-encoded and in PEM format: `{{ pem .Secrets.rsa_cert.Public }}`
* Inject a symmetric key in hex format: `{{ hex .Secrets.secret_aes_key }}`

## Manifest:Users
The optional entry `Users` defines user credentials and permissions for authentication and access control.
Each user is authenticated via a client certificate. The certificate needs to be specified as a PEM-encoded self-signed X.509 certificate.
Users can [update a manifest]({{< ref "docs/workflows/update-manifest.md">}}) and [read or write secrets]({{< ref "docs/workflows/managing-secrets.md" >}}).

```javascript
{
    //...
    "Users": {
        "alice": {
            "Certificate": "-----BEGIN CERTIFICATE-----\nMIIFPjCCA..."
        },
        "bob": {
            "Certificate": "-----BEGIN CERTIFICATE-----\nMIIFP..."
        }
    }
}
```
When verifying certificates in this context, Marblerun ignores their `issuer`, `subject`, and `expiration date` fields. Thus, users cannot lock themselves out through expired certificates.

Use OpenSSL to generate a compatible certificate.

```bash
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes -keyout admin_private.key -out admin_certificate.crt
```

Use the following command to preserve newlines correctly:

```bash
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' admin_certificate.pem
```

## Manifest:RecoveryKeys

The optional entry `RecoveryKeys` holds PEM-encoded RSA public keys which can be used to recover a failed Marblerun deployment. (The process of recovering a Marblerun instance is described in our [recovery chapter]({{< ref "docs/features/recovery.md" >}})). So far, only one public key entry is supported in the current release of Marblerun.

```javascript
{
    //...
    "RecoveryKeys":
    {
        "recoveryKey1": "-----BEGIN PUBLIC KEY-----\nMIIBpTANBgk..."
    }
}
```

This key can be generated with the help of OpenSSL.

```bash
openssl genrsa -out private_key.pem 4096
openssl rsa -in private_key.pem -outform PEM -pubout -out public_key.pem
```

Use the following command to preserve newlines correctly:

```bash
awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' public_key.pem
```


## Manifest:TLS

The TLS entry holds a list of tags which can be used in a Marble's definition. Each tag can define multiple `Incoming` and `Outgoing` connections. To elevate the connection between two marbles to TLS, the client needs to set the server under `Outgoing` and the server needs to define its service under `Incoming`.

Outgoing connections are defined by `Port` and `Addr`. For `Addr`, you can use both IP addresses and domains, e.g., the DNS names of other services.

Incoming connections are defined by `Port`. For services used by external clients, you must disable client authentication by setting `DisableClientAuth` to `true` and set `Cert`. Use the name of a certificate defined in the [Secrets section](#manifestsecrets).

```javascript
{
    //...
    "TLS":
    {
        "frontend_tls_1": {
            "Outgoing": [
                {
                    "Port": "8080",
                    "Addr": "service.name"
                },
                {
                    "Port": "4443",
                    "Addr": "10.111.37.164"
                }
            ],
            "Incoming": [
                {
                    "Port": "8443"
                },
                {
                    "Port": "8080",
                    "Cert": "rsa_cert",
                    "DisableClientAuth": true
                }
            ]
        },
        "backend_first_tls": {
            // ...
        }
    }
}
```

## Manifest:Infrastructures

Future versions of Marblerun will allow you to define certain trusted infrastructures and hardware configurations under `Infrastructures`.
