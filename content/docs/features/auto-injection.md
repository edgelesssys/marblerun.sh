---
title: "Auto-injection"
draft: false
weight: 4
---
# Automatic Marble Injection

Services Meshes are typically implemented using a sidecar proxy as we've explained in our [concepts]({{< ref "docs/getting-started/concepts.md#marblerunapproach" >}}) section.
Facilitating their deployment they often provide a feature known as "proxy injection" which automatically adds the data plane proxy to pods for workloads inside the mesh.
In Kubernetes, this usually implemented using a [MutatingAdmissionWebhook](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook).
Marblerun injects the data-plane logic directly into the application logic running inside secure enclaves, hence, there is no need for a "proxy-injection".
However, our data plane requires its deployment-specific configuration to be provided through the Kubernetes resource definitions.
Similar to the proxy-injection, we make use of an admission controller to automatically inject this configuration for workloads inside the Marblerun mesh.
It optionally injects [tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) and [resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) for our SGX device plugin.
For more information about SGX device plugin see the [SGX Device Plugin]({{< ref "docs/getting-started/sgx-device-plugin.md" >}}) section.

See [Add a Service]({{< ref "docs/getting-started/concepts.md#marblerunapproach" >}}) for a walkthrough of how to use this feature in practice.

You can enable a namespace for auto-injection using the Marblerun CLI:

```bash
marblerun namespace add NAMESPACE [--inject-sgx]
```

This will add the label `marblerun/inject=enabled` to the chosen namespace and allow the admission webhook to intercept the creation of deployments, pods, etc. in that namespace.
If the `--inject-sgx` flag is set this will additionally add the label `marblerun/inject-sgx=enabled`.
See [SGX device injection](#sgx-device-injection) for more information about this label.

##  Marbletype Label

When defining your [manifest]({{< ref "docs/tasks/define-manifest.md" >}}) you specify which Marbles should be part of your confidential mesh.
Currently, there is no direct link between the Marble in the manifest and the Kubernetes app.
Hence, our admission controller needs to know which Marble a deployment/pod/etc. corresponds to.
To that end, Marblerun requires one addition to your resource description in the form of the `marblerun/marbletype` label.
It references the Marble in your Manifest and the value should reflect the name of the Marble:

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

By default, any pods trying to deploy in a namespace added to a Marblerun mesh will be rejected, unless they possess the label `marblerun/marbletype`.

## Injected Environment Variables

The webhook will inject the following environment variables into each container of a pod:

* `EDG_MARBLE_TYPE`:  The value of the `marblerun/marbletype` label
* `EDG_MARBLE_COORDINATOR_ADDR`:  The address of the Marblerun coordinator running on the cluster
* `EDG_MARBLE_DNS_NAMES`:  DNS names of the pod derived from marbletype and namespace: `marbletype, marbletype.namespace, marbletype.namespace.svc.cluster.local`
* `EDG_MARBLE_UUID_FILE`:  The mounted UUID of the Marble

If an environment variable is already set before the webhook handles the creation request, the variable will not be overwritten and the custom value is used instead.

## SGX Device Injection

Your confidential apps need access to the SGX device inside the container they are running in.
See our section on the [SGX device plugin]({{< ref "docs/getting-started/sgx-device-plugin.md" >}}) for more information on how SGX is managed in Kubernetes.
You can deploy your own SGX device plugin in your cluster or let Marblerun take care of it.
By default, Marblerun will check whether an SGX device plugin is already present in the cluster and only install one if that is not the case.
Using the flag `--inject-sgx` to add a namespace to the Marblerun mesh will additionally inject all pods starting in that namespace with the required configuration for the SGX device plugin that ships with Marblerun.
