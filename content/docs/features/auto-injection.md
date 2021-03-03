---
title: "Auto-injection"
draft: false
weight: 4
---
# Integration with Kubernetes

Marblerun provides its data-plane configuration through Kubernetes resource definitions. For this, like normal service meshes, Marblerun uses Kubernetes' [admission controllers](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook).

Marblerun optionally injects [tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) and [resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for its SGX device plugin. See the [SGX Device Plugin]({{< ref "docs/getting-started/sgx-device-plugin.md" >}}) section for more information.
See the [Add a Service]({{< ref "docs/getting-started/concepts.md#marblerunapproach" >}}) section for how to use this feature in practice. You can enable auto-injection of the SGX device plugin as follows using the Marblerun CLI:

```bash
marblerun namespace add NAMESPACE [--inject-sgx]
```

This will add the label `marblerun/inject=enabled` to the chosen namespace and allow the admission webhook to intercept the creation of deployments, pods, etc. in that namespace.
The flag `--inject-sgx` sets the label `marblerun/inject-sgx=enabled`. The label is explained in more detail in the [SGX device injection](#sgx-device-injection) section.

## The Marbletype label

In Marblerun, marbles (i.e, secure enclaves) are defined in the [manifest]({{< ref "docs/tasks/define-manifest.md" >}}). You need to reference marbles in your Kubernetes resource description as follows using the `marblerun/marbletype` label:

```javascript
{
    "Marbles": {
        "voting-svc": {
    // ...
```

```yaml
  labels:
    marblerun/marbletype: voting-svc
```

Pods without a valid `marblerun/marbletype` label are rejected in a namespace with. By default, any pods trying to deploy in a namespace with enabled auto injection (i.e., `marblerun/inject=enabled`).

## Injected environment variables

The webhook will inject the following environment variables into each container of a pod:

* `EDG_MARBLE_TYPE`:  The value of the `marblerun/marbletype` label
* `EDG_MARBLE_COORDINATOR_ADDR`:  The address of the Marblerun coordinator running on the cluster
* `EDG_MARBLE_DNS_NAMES`:  DNS names of the pod derived from marbletype and namespace: `marbletype, marbletype.namespace, marbletype.namespace.svc.cluster.local`
* `EDG_MARBLE_UUID_FILE`:  The mounted UUID of the Marble

If an environment variable is already set before the webhook handles the creation request, the variable will not be overwritten and the custom value is used instead.
