---
title: "Coordinator"
date: 2020-11-19T16:30:52+01:00
draft: false
weight: 3
---

# Coordinator

The Coordinator is Marblerun's control plane.
It communicates with the Marble's data plane through gRPC and provides an HTTP-REST interface on the client-side.
The Coordinator can be configured with several environment variables:

* `EDG_COORDINATOR_MESH_ADDR`: The listener address for the gRPC server
* `EDG_COORDINATOR_CLIENT_ADDR`: The listener address for the HTTP server
* `EDG_COORDINATOR_DNS_NAMES`: The DNS names for the cluster's root certificate
* `EDG_COORDINATOR_SEAL_DIR`: The file path for storing sealed data

The Coordinator clients can be divided into two major groups.

* The owners/providers/administrators who need to interact with the Coordinator for deploying their confidential application and administrative tasks
* The users/customers who use the Coordinator for remote attestation and establishing trust with the application

The [Client API]({{< ref "docs/reference/coordinator" >}}) serves both use-cases with a compact REST-API.
