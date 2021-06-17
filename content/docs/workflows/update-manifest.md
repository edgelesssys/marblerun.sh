---
title: "Updating a Manifest"
date: 2021-01-19T15:33:00+01:00
draft: false
weight: 6
---

# Updating a Manifest
In order to ensure the confidentiality of a deployed application, Marblerun uses a Manifest to define the software packages and the infrastructure your deployment uses. To verify that your deployment has not been altered, the Manifest is usually set in stone after it was set to ensure no one can alter with your cluster.

Yet, updates play an important role to ensure your software stays secure. To avoid having to redeploy your application from scratch, Marblerun allows uploading a separate "Update Manifest" which increases the minimum `SecurityVersion` of one or multiple already deployed packages. After such an update is performed, an old version of a defined software package cannot be loaded anymore under the current Manifest.

## Requirements
In order to deploy an Update Manifest, the original Manifest, you need to be in possession of a certificate/private key pair which has been defined in the `Users` section of the original Manifest, as described in ["Defining a Manifest"]({{< ref "docs/workflows/define-Manifest.md#manifestmarbles" >}}).

If no administrator has been initially set up, no Manifest Updates can be applied.

## Defining an Update Manifest
The format of an Update Manifest follows the syntax of the original Manifest, though it only expects to contain a package name and a new `SecurityVersion` value set for it.

For example, the current `Packages` section of your original Manifest looks like this:

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

If you now want to update the minimum required version for `pkg1`, the complete definition for the update Manifest just needs to be as short as this example:


```javascript
{
    "Packages": {
        "pkg1": {
            "SecurityVersion": 5
        }
    }
}
```

Please do not define other values except than `SecurityVersion` value for a package, as Marblerun will refuse to accept such an Update Manifest.

Also, if an Update Manifest was already set and you want to deploy another update on top of it too, you can! Just make sure that the new Update Manifest contains each specified package as the old one does, and that the `SecurityVersion` is indeed higher than defined in the previous Update Manifest, as downgrades are not supported for security reasons.

## Deploying an Update Manifest
Similar to other operations, an Update Manifest can be deployed e.g. with the help of the CLI. Note that for this operation, you need to specify one of your defined `Users` certificates as a TLS client certificate, combined with the according private key.

This operation can be performed in the following way:

```bash
marblerun manifest update update-manifest.json $MARBLERUN --cert=admin-cert.pem --key=admin-key.pem --era-config=era.json
```

If everything went well, no message will be returned and your Marblerun logs should highlight that an Update Manifest has been set. And if something went wrong, the API endpoint will return an error message telling you what happened. If you receive `unauthorized user` back, it means Marblerun either received no client certificate over the TLS connection, or you used the wrong certificate.

## Effects of an Update Manifest
When a Manifest has been updated, the coordinator will generate new certificates which your Marbles will receive upon the next startup. Also, if you are trying to launch Marbles based on packages containing the old `SecurityVersion`, they will refuse to run (unless you are running in SGX Simulation or non-Enclave mode). However, so far currently running Marble will continue to run and will be able to authenticate each other as long as they are still running, so if you need to enforce an update, make sure to kill the Marbles on your host and restart them.
