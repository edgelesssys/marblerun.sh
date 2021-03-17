---
title: "Marbles"
date: 2020-11-19T16:33:26+01:00
draft: false
weight: 4
---

# Marbles

Marbles represent the data plane in Marblerun and run your actual application code in secure enclaves within otherwise normal Docker containers. Marbles communicate with the Coordinator via gRPC over TLS. See the [Add a Service]({{< ref "docs/tasks/add-service.md" >}}) section on how to build and configure a Marble.
