---
title: Marblerun - The service mesh for Confidential Computing
claim: The service mesh for Confidential&nbsp;Computing
description:
  "**Marblerun** makes it easy to deploy, scale, and verify your SGX-based apps on vanilla Kubernetes. Think Istio/Consul/Linkerd for Confidential Computing. It's open source, written in Go, and truely cloud native."
buttons:
  get_started:
    caption: Get Started →
    url: '/docs/getting-started/quickstart/'
  community:
    caption: Join the Community
    url: '/community/'
features_list:
  items:
    - title: Everything always encrypted (even&nbsp;at&nbsp;runtime)
      icon: fas fa-lock
      description:
        All services run in secure enclaves; your data and code are encrypted even at runtime. Between enclaves, data is transmitted via mTLS. 
    - title: End-to-end verifiability for the whole cluster
      icon: fas fa-file-signature
      description:
        Get cryptographic proof that the topology of your cluster adheres to a Manifest defined in simple JSON.
    - title: Keep using your existing tools and stacks
      icon: fas fa-tools
      description:
        Despite using the latest Confidential Computing tech, Marblerun works frictionless with [K8s](https://kubernetes.io/), [Helm](https://helm.sh/) and conventional services meshes. 
    - title: Cloud native and cloud agnostic
      icon: fas fa-cloud
      description:
        Marblerun is written in Go and uses standards like gRPC and REST. It scales and secures your apps in any cloud that has Intel SGX - like [Azure](https://azure.microsoft.com/en-us/solutions/confidential-compute/).
    - title: Deploy your first app in minutes 
      icon: fas fa-shipping-fast
      description:
        Usability and simplicity (and security of course!) are our guiding principles. Porting and deploying existing distributed Go apps only takes a few [simple steps](/docs/getting-started/quickstart).
    - title: Open source and open standards
      icon: fab fa-github
      description:
        Marblerun is open source and builds upon the industry standard [Open Enclave](https://openenclave.io/sdk/) and [Edgeless RT](https://github.com/edgelesssys/edgelessrt). In the future, it will also support [Graphene](https://github.com/oscarlab/graphene).
---