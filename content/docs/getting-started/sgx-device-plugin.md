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


If you are using a CC-enlightened, managed Kubernetes cluster, you will usually already have an SGX device plugin installed.
For example, creating a confidential computing cluster on AKS has a preconfigured SGX device plugin. See [the quickstart guide](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-aks-get-started) on how to get started.


## Manually deploying an SGX device plugin

For different reasons, you may want to deploy the device plugin manually. Intel provides [a guide](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/sgx_plugin/README.html#installation) to install their SGX plugin.
In any case, you will need to adjust your deployments to request the SGX resources provided by the plugin:

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
Note, that every plugin uses its own way of injecting SGX resources into deployments. Please refer to the documentation for your plugin of choice. This is an example of the Intel plugin.

Marblerun supports [automatic injection]({{< ref "docs/features/auto-injection.md" >}}) of those values, provided your used plugin is supported by Marblerun.

{{<note>}}
Currently supported plugins are:
* [Intel](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/sgx_plugin/README.html) using `sgx.intel.com/epc`
* [Azure](https://github.com/Azure/aks-engine/blob/master/docs/topics/sgx.md#deploying-the-sgx-device-plugin) using `kubernetes.azure.com/sgx_epc_mem_in_MiB`

If you are using a different plugin please let us know, so we can add support!
{{</note>}}


## Out-of-process attestation

Intel SGX supports two modes for obtaining remote attestation quotes:
* In-process: The software generating the quote is part of the enclave application
* Out-of-process: The software generating the quote is not part of the actual enclave application. This requires the Intel SGX Architectural Enclave Service Manager (AESM) to run on the system

While Marbles build with [Ego]({{< ref "docs/tasks/build-service-ego" >}}) perform in-process attestation, other frameworks, such as [Graphene]({{< ref "docs/tasks/build-service-graphene" >}}), use out-of-process attestation.
If your confidential application uses out-of-process attestation, you will need to expose the AESM device to your container.

You can follow [the AKS guide](https://docs.microsoft.com/en-us/azure/confidential-computing/confidential-nodes-out-of-proc-attestation) to make your deployments able to use AESM for quote generation. Note, that in this case, your Kubernetes nodes need the AESM service installed. See the [Intel installation guide](https://download.01.org/intel-sgx/sgx-linux/2.12/docs/Intel_SGX_Installation_Guide_Linux_2.12_Open_Source.pdf) for more information.