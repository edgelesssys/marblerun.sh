---
title: "Overview"
draft: false
weight: 1
---

# Overview

Marblerun is designed to be flexible and adjustable to different deployment scenarios and use cases. The following questions are essential.

{{<note>}}
Marblerun without Intel SGX can be deployed on arbitrary bare-metal machines or VMs. Please see
our [quickstart]({{< ref "docs/getting-started/quickstart.md" >}}) guide for the details.
{{</note>}}

## Cloud or on-prem?

To run Marblerun in the cloud, follow this [guide]({{< ref "docs/deployment/cloud.md" >}}).

To deploy Marblerun somewhere else, e.g., on-prem, use this [guide]({{< ref "docs/deployment/on-prem.md" >}}).

## Kubernetes or standalone?

Marblerun is designed to orchestrate confidential apps on Kubernetes. The simple deployment steps are described in this [guide]({{< ref "docs/deployment/kubernetes.md" >}}).

Marblerun can also very well be used to manage confidential workloads indepently of Kubernetes, as is described in this [guide]({{< ref "docs/deployment/standalone.md" >}}).
