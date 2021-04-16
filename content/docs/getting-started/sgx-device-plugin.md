---
title: "SGX device plugin"
draft: false
weight: 6
---

# SGX device plugin on Kubernetes

Kubernetes manages hardware resources like Intel SGX through its [device plugin framework](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/).
The SGX device plugin can either be deployed manually or as a DaemonSet in the cluster. Different vendors provide open-source device plugins for SGX:

* [Intel](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/sgx_plugin/README.html)
* [Azure](https://github.com/Azure/aks-engine/blob/master/docs/topics/sgx.md#deploying-the-sgx-device-plugin)
* [Alibaba Cloud](https://github.com/AliyunContainerService/sgx-device-plugin)


## Creating a cluster with an SGX device plugin

If you are creating your SGX enabled kubernetes cluster using a service offered by a cloud provider, you will usually not need to choose your own.
For example following [the quickstart guide](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-aks-get-started) for a confidential computing cluster on AKS will create the cluster including SGX device plugin.


## Out-of-process attestation

Applications running in an enclave require a generated quote to perform remote attestation. To do this Intel SGX supports two modes:
* In-process: The software generating the quote is part of the enclave application
* Out-of-process: The software generating the quote is not part of the actual enclave application, this requires the Intel SGX Architectural Enclave Service Manager (AESM) to run on the system

While Marbles build with [Ego]({{< ref "docs/tasks/build-service-ego" >}}) perform in-process attestation, other frameworks, such as [Graphene]({{< ref "docs/tasks/build-service-graphene" >}}), use out-of-process attestation.
If you are planning to deploy your confidential application on kubernetes using out-of-process attestation, you will need to expose AESM to your application in some way.

For clusters created on AKS you can follow [this guide](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-out-of-proc-attestation) to make your deployments able to use AESM for quote generation.


## Manually deploying an SGX device plugin

For different reasons, you may want to deploy the device plugin manually. Intel provides [a guide](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/sgx_plugin/README.html#installation) to install their SGX plugin, however you may use any implementation exposing the SGX resource on the cluster.
You will need to adjust your deployments to request the SGX resource provided by the plugin:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oe-deployment
spec:
  selector:
    matchLabels:
      app: oe-app
  replicas: 1
  template:
    metadata:
      labels:
        app: oe-app
    spec:
      tolerations:
      - key: sgx.intel.com/epc
        operator: Exists
        effect: NoSchedule
      containers:
      - name: <image_name>
        image: <image_reference>
        command: <exec>
        resources:
          limits:
            sgx.intel.com/epc: 10
```
Note that in this case, the plugin by Intel is used.
Marblerun supports [automatic injection]({{< ref "docs/features/auto-injection.md" >}}) of those values, provided your used plugin is supported by Marblerun.

{{<note>}}
Currently supported plugins are:
* [Intel](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/sgx_plugin/README.html) using `sgx.intel.com/epc`
* [Azure](https://github.com/Azure/aks-engine/blob/master/docs/topics/sgx.md#deploying-the-sgx-device-plugin) using `kubernetes.azure.com/sgx_epc_mem_in_MiB`

If you are using a different plugin please let us know, so we can add support!
{{</note>}}
