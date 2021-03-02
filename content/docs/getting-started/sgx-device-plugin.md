---
title: "SGX Device Plugin"
draft: false
weight: 6
---

# Kubernetes SGX Device Plugin

Kubernetes manages hardware resources as Intel SGX through a [device plugin framework](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/).
On Kubernetes, exposing SGX to confidential workloads requires an SGX device plugin running on the nodes they are scheduled on.
The device plugin can either be deployed manually or as a DaemonSet in the cluster. You can write your own device plugin, however, there are already a couple of public open-source SGX Device Plugins from different vendors:

* [Intel](https://intel.github.io/intel-device-plugins-for-kubernetes/cmd/sgx_plugin/README.html)
* [Azure](https://github.com/Azure/aks-engine/blob/master/docs/topics/sgx.md#deploying-the-sgx-device-plugin)
* [Alibaba Cloud and Ant Financia](https://github.com/AliyunContainerService/sgx-device-plugin)

When deploying Marblerun it will check whether an SGX device plugin is already running and optionally deploy one if that is not the case.

```bash
marblerun install [--no-sgx-device-plugin]
```

When writing this feature, there was no official device plugin available from Intel so we decided to use Azure's plugin.
Scheduling pods to TEE enabled hardware requires the following additions to your Kubernetes resource definitions:

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
      - key: kubernetes.azure.com/sgx_epc_mem_in_MiB
        operator: Exists
        effect: NoSchedule
      containers:
      - name: <image_name>
        image: <image_reference>
        command: <exec>
        resources:
          limits:
            kubernetes.azure.com/sgx_epc_mem_in_MiB: 10
```

See our [auto-injection]({{< ref "docs/features/auto-injection.md" >}}) feature for more information on how we inject these values automatically.
Note that this plugin is not Azure-specific and can be deployed on any cluster with SGX hardware.
We might switch to the official Intel device plugin anytime in the future.
