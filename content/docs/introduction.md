---
title: "Introduction"
date: 2020-11-14T16:28:16+05:30
draft: false
aliases:
  - /docs/
weight: 1
---

# Introduction

![logo](/img/mr_logo.svg)

Marblerun is a framework for creating distributed confidential-computing apps.

Build your confidential microservices with [EGo or another runtime]({{< ref "docs/features/runtimes.md" >}}), distribute them with Kubernetes on an SGX-enabled cluster, and let Marblerun take care of the rest. Deploy end-to-end secure and verifiable AI pipelines or crunch on sensitive big data in the cloud. Confidential computing at scale has never been easier.

Marblerun guarantees that the topology of your distributed app adheres to a Manifest specified in simple JSON. Marblerun verifies the integrity of services, bootstraps them, and sets up encrypted connections between them. If a node fails, Marblerun will seamlessly substitute it with respect to the rules defined in the Manifest.

To keep things simple, Marblerun issues one concise remote-attestation statement for your whole distributed app. This can be used by anyone to verify the integrity of your distributed app.

## Key features

* Authentication and integrity verification of microservices wrt. the Manifest :lock:
* Secrets management for microservices :key:
* Provisioning of certificates, configurations, and parameters :package:
* Remote attestation of the entire cluster :globe_with_meridians:

## Overview

Logically, Marblerun consists of two parts, the control plane called *Coordinator* and the data plane called *Marbles*.
The Coordinator needs to be deployed once in your cluster and the Marble layer needs to be integrated with each service.
Marblerun is configured with a simple JSON document called the *Manifest*.
It specifies the topology of the distributed app, the infrastructure properties, and provides configuration parameters for each service.

![overview](/img/overview.svg)
