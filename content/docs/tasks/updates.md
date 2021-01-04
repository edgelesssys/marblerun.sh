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
