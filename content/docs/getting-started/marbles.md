---
title: "Marbles"
date: 2020-11-19T16:33:26+01:00
draft: false
weight: 6
---

# Marbles

Marbles represent the data plane in Marblerun and run your actual application code in secure enclaves within otherwise normal Docker containers. Marbles communicate with the Coordinator via gRPC over TLS. See the [Add a Service]({{< ref "docs/tasks/add-service.md" >}}) section on how to build a Marble.

Marbles can be configured with several environment variables.

* `EDG_MARBLE_COORDINATOR_ADDR`: The Coordinator's address
* `EDG_MARBLE_TYPE`: The Marble's Package
* `EDG_MARBLE_DNS_NAMES`: The DNS names in the Marble's Certificate
* `EDG_MARBLE_UUID_FILE`: The file path for storing the Marble's UUID, needed for restart persistence.

