---
title: "Supported Runtimes"
draft: false
weight: 4
---

# Supported Runtimes

Marblerun strives to be runtime-agnostic. Currently supported runtimes are described below. More will follow in future.

## EGo
[EGo](https://github.com/edgelesssys/ego) is the preferred way for writing confidential Go applications from scratch as well as porting existing ones. Usage is very similar to conventional Go programming. Continue [here]({{< ref "docs/tasks/build-service-ego.md" >}}) to use it with Marblerun.

## Edgeless RT
With [Edgeless RT](https://github.com/edgelesssys/edgelessrt) you can create confidential C++ applications with a low TCB. Please follow the build instructions provided [here](https://github.com/edgelesssys/marblerun/blob/master/samples/helloc%2B%2B) to use it with Marblerun.

## Graphene
[Graphene](https://grapheneproject.io/) is a popular choice for wrapping unmodified applications into enclaves.
This approach commonly known as "lift and shift" facilitates the process of bringing existing applications into the confidential space.
Graphene further adds support for dynamically linked libraries and multi-process applications in SGX.
Running a Graphene app with Marblerun requires [minor changes]({{< ref "docs/tasks/build-service-graphene.md" >}}) to its manifest.
