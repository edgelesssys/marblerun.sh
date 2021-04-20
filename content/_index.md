---
title: Marblerun - The service mesh for confidential computing
claim: The service mesh for confidential computing
description:
  "**Marblerun** makes it easy to deploy, scale, and verify your SGX-based apps on vanilla Kubernetes. Think Istio/Consul/Linkerd for [confidential computing](https://confidentialcomputing.io/). It's open source, written in Go, and truely cloud native."
buttons:
  get_started:
    caption: Get started →
    url: '/docs/getting-started/quickstart/'
  community:
    caption: Join the community
    url: '/community/'
news:
  title: "Announcing [Marblerun 0.3.0](/docs/changelog/)"
  content: "Adds support for [Graphene-based services](/docs/examples/graphene/) and an [easy-to-use CLI](/docs/getting-started/cli/)."
features_list:
  items:
    - title: Everything always encrypted
      icon: fas fa-lock
      description:
        All services run in secure enclaves; your data and code are encrypted even at runtime. Between enclaves, data is transmitted via mTLS.
    - title: End-to-end verifiability
      icon: fas fa-file-signature
      description:
        Get cryptographic proof that the topology of your cluster adheres to a Manifest defined in simple JSON.
    - title: Keep using your existing tools
      icon: fas fa-tools
      description:
        Despite using the latest confidential-computing tech, Marblerun works frictionless with [K8s](https://kubernetes.io/), [Helm](https://helm.sh/) and normal services meshes like [Istio](https://istio.io/) or [Linkerd](https://linkerd.io/).
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
        Marblerun is open source and builds upon the industry standard [Open Enclave](https://openenclave.io/sdk/). It has support for [EGo](https://www.ego.dev/) and [Graphene](https://github.com/oscarlab/graphene) based services.
blog_list:
  items:
    - title: "[Why confidential computing?](https://edgelesssys.medium.com/why-we-need-a-service-mesh-for-confidential-computing-part-1-3-28f4bd6df679)"
      description: We give an intro to the concept of confidential computing.
    - title: "[Why services meshes?](https://edgelesssys.medium.com/why-we-need-a-service-mesh-for-confidential-computing-part-2-3-c417fa581ef5)"
      description: We give intro to the service mesh concept and discuss crucial properties in the context of confidential computing.
    - title: "[An introduction to Marblerun](https://edgelesssys.medium.com/why-we-need-a-service-mesh-for-confidential-computing-part-3-3-ffc00b2c3508)"
      description: We describe key features and the architecture of Marblerun.
    - title: "[Running Graphene on Marblerun](https://medium.com/edgelesssystems/marblerun-now-supports-graphene-thus-your-favorite-programming-language-a8b8a36787a0)"
      description: We give an intro to Marblerun's support for Graphene-based applications.
---
