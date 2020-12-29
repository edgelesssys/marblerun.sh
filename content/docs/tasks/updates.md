---
title: "Updating a deployment"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 7
---

# Updating a deployment

The following gives a walkthrough of typical deployment updates in a Kubernetes cluster and how to handle them with Marblerun.

## Updating to a new Marblerun version

When updating to a new Marblerun version, updates to both the control plane and data plane components may be required.

### Updating the Coordinator

Updating the Coordinator follows the regular steps for [updating a deployment in Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment).

```bash
kubectl -n marblerun set image deployment/marblerun-coordinator coordinator=ghcr.io/edgelesssys/coordinator:v0.2.0 --record
```

You can also use Helm to upgrade the image. Note that Helm requires you to pass all the flags with `upgrade` that you set during the initial deployment.

```bash
helm upgrade marblerun-coordinator edgeless/marblerun-coordinator \
    -n marblerun \
    --set coordinator.coordinatorImageVersion=v0.2.0
```

If the Coordinator is rescheduled on the same host as before, it will continue running with the same Manifest as before.
However, if the Coordinator gets rescheduled to another node during the updating process you need to perform the [recovery step]({{< ref "docs/features/recovery.md" >}}).

The Marbles won't be affected by the Coordinator update and will continue running.
New Marbles that are started can interact with existing ones.
If the Coordinator version update also affects the data plane or the Marble bootstrapping process, you will need to restart your Marbles.

```bash
kubectl rollout restart deployment your_deployment_name
```

**Note:** when updating the Coordinator image, you need to be careful about your client's configuration.
If you specified a `UniqueID`, your clients won't accept the new Coordinator version.
See our [remarks on Manifest values](#manifest-values) for more information.

### Updating Marbles

The tight integration of Marblerun's data plane with your application requires a rebuild of your application and the corresponding containers.
Note that you only need to update if there have been changes affecting the data plane.
The process is similar to a regular update of your application, as is described in the following.

## Updating your confidential application

Updating your application is straightforward with Marblerun.
You can roll out the new version similar to a regular [deployment update](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment).
The Coordinator will manage the attestation and bootstrapping of your new version.
Notably, nothing changes on the client-side of Marblerun, the version update is transparent and adherent to the Manifest in effect.

### Manifest values

Making updates work seamlessly requires some attention when defining enclave software-packages in the `Packages` section of the Manifest.
Take the following Manifest as example.

```json
{
    "Packages": {
        "pkg0": {
            "UniqueID": "6b2822ac2585040d4b9397675d54977a71ef292ab5b3c0a6acceca26074ae585",
            "Debug": false
        },
        "pkg1": {
            "SignerID": "43361affedeb75affee9baec7e054a5e14883213e5a121b67d74a0e12e9d2b7a",
            "ProductID": 42,
            "SecurityVersion": 1,
            "Debug": false
        }
    }
}
```

The properties `UniqueID`, `SignerID`, `ProductID`, and `ProductID` are described in more detail [here]({{< ref "docs/tasks/define-manifest.md#manifestpackages" >}}).
In the case of our example, `pkg0` is identified through `UniqueID`. Since `UniqueID` is the cryptographic hash of the enclave software-package, this means that `pkg0` cannot be updated. (That is, because any update to the package will change the hash.)

In contrast, `pkg1` is identified through the triplet `SignerID`, `ProductID`, and `SecurityVersion`. `SignerID` cryptographically identifies the vendor of the package; `ProductID` is an arbitrary product ID chosen by the vendor, and `SecurityVersion` is the security-patch level of the product. See [here]({{< ref "docs/tasks/add-service.md#step-21-define-the-enclave-software-package" >}}) on how to get these values for a given service.

Future versions of Marblerun will accept any `SecurityVersion` that is equal or higher than the one specified in `Packages` for a given combination of `SignerID` and `ProductID`. This way, updates to packages can be made without having alter the Manifest.
