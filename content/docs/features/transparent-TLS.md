---
title: "Transparent TLS"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 3
---

# Transparent TLS

Authenticated and encrypted connections between services are essential for the security and verifiability of confidential applications. These properties are provided by mutual TLS authentication (mTLS). Normally, the applications inside the Marbles must support mTLS, be configured correctly, and be provisioned with the necessary secrets.

Transparent TLS (TTLS) can wrap any connection in TLS on the Marblerun layer. Marblerun adds secure communication to your cluster even if your application does not support the required TLS features. Just define the desired [connections in the Manifest]({{< ref "docs/workflows/define-manifest.md#manifesttls" >}}).

TTLS is currently available with [EGo Marbles]({{< ref "docs/building-services/ego.md" >}}). Other [runtimes]({{< ref "docs/features/runtimes.md" >}}) will be supported in future.

## Authentication and credentials
By default the Marble's credentials are automatically configured. Connections between two Marbles are mutually authenticated.

You can use custom credentials defined in the Manifest's secrets. This can be useful when connecting from outside the cluster, to always serve the same certificate.
