---
title: "Auto-injection"
draft: false
weight: 4
---
# Auto-injection

By default a Marblerun installation ships with a kubernetes [MutatingAdmissionWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook). <br>
This admission controller monitors selected namespaces of the cluster and controlls the creation of pods in those namespaces, by preventing pods with missing labels from starting and injecting allowed pods with additional environment variables for the Marblerun mesh, as well as the option to inject [tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) for sgx enabled nodes.

To enable a namespace for auto-injection by the webhook, one can use the Marblerun cli:

```bash
marblerun namespace add NAMESPACE [--inject-sgx]
```

This will add the label `marblerun/inject=enabled` to the chosen namespace and allow the webhook to intercept the creation of pods in that namespace.
If the `--inject-sgx` flag is set this will additionaly add the label `marblerun/inject-sgx=enabled`.

##  Marbletype label

By default any pods trying to deploy in a namespace added to a Marblerun mesh will be rejected, unless they posses the label `marblerun/marbletype`.
This value should reflect the name of the marble defined in the manifest.


## Injected Environment Variables

The webhook will inject the following environment variables into each container of a pod:

*   EDG_MARBLE_TYPE:  The value of the `marblerun/marbletype` label
*   EDG_MARBLE_COORDINATOR_ADDR:  The address of the Marblerun coordinator running on the cluster
*   EDG_MARBLE_DNS_NAMES:  DNS names of the pod derived from marbletype and namespace: `marbletype, marbletype.namespace, marbletype.namespace.svc.cluster.local`
*   EDG_MARBLE_UUID_FILE:  The UID of the pod

If an environemt variable is already set before the webhook handles the creation request, the variable will not be overwritten and the custom value is used instead.

## SGX-Tolerations Injection

Using the flag `--inject-sgx` to add a namespace to the Marblerun mesh will additionally inject all pods starting in that namespace with tolerations for SGX enabled nodes. For more information see [Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/).
