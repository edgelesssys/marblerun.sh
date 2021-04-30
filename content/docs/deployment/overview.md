---
title: "Overview"
draft: false
weight: 1
---

# Deployment overview

Marblerun was designed to be flexible and adjustable to different deployment scenarios.
It's cloud-agnostic and can run on different platforms and Cloud Service Providers (CSPs).
To give you the best guidelines and options for your deployment we split this section based on two key questions:

## Cloud vs. On-premises

The first question toward a confidential computing deployment is the type of hosting.

Do you want to deploy Marblerun on your on-premises hardware?
We have created [a summary of the prerequisites and necessary steps]({{< ref "docs/deployment/on-prem.md" >}}) to create a production environment for your confidential application.

If you prefer to use a CSP, we have assembled [a list of confidential computing offerings]({{< ref "docs/deployment/cloud.md" >}}) and how to use them with Marblerun.

## Standalone vs Kubernetes

The second question is the type of orchestration you want to use for your application.

The simplest way of using Marblerun is in a Kubernetes cluster.
We have created a [guide]({{< ref "docs/deployment/kubernetes.md" >}}) on the prerequisites of using SGX in a Kubernetes cluster and the process of deploying Marblerun in your cluster.

Alternatively, you can also use Marblerun standalone.
We have created a [guide]({{< ref "docs/deployment/standalone.md" >}}) on how to configure and run the Coordinator control-plane and how to configure the data-plane in your workloads.
