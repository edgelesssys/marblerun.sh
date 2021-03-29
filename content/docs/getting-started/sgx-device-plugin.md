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

Marblerun checks if an SGX device plugin is already running and deploys Azure's plugin otherwise.

{{<note>}}
The Azure SGX plugin is not tied to Azure. We may however switch to Intel's device plugin in the future.
{{</note>}}

## Manually deploying an SGX device plugin

For different reasons, you may want to deploy the device plugin manually. This requires two steps. First, add `tolerations` and `resources` to your Kubernetes deployment spec as outlined below.

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

Note that in this case, the plugin from Azure is used. Second, install Marblerun using the `--no-sgx-device-plugin` flag:

```bash
marblerun install [--no-sgx-device-plugin]
```
