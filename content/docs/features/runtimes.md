---
title: "Supported runtimes"
draft: false
weight: 4
---

# Supported runtimes

Marblerun strives to be runtime-agnostic. Currently supported runtimes are described below. More will follow in future.

## EGo
[EGo](https://github.com/edgelesssys/ego) is the preferred way for writing confidential Go applications from scratch as well as porting existing ones. Usage is very similar to conventional Go programming. Continue [with our building a service with EGo hands-on]({{< ref "docs/building-services/ego.md" >}}) to use it with Marblerun.

## Edgeless RT
With [Edgeless RT](https://github.com/edgelesssys/edgelessrt) you can create confidential C++ applications with a low TCB. Please follow the build instructions provided [in our C++ sample](https://github.com/edgelesssys/marblerun/blob/master/samples/helloc%2B%2B) to use it with Marblerun.

## Graphene
[Graphene](https://grapheneproject.io/) is a popular choice for wrapping unmodified applications into enclaves.
This approach commonly known as "lift and shift" facilitates the process of bringing existing applications into the confidential space.
Graphene further adds support for dynamically linked libraries and multi-process applications in SGX.
Running a Graphene app with Marblerun requires [minor changes]({{< ref "docs/building-services/graphene.md" >}}) to its manifest.

## Occlum
[Occlum](https://github.com/occlum/occlum) is another popular solution Graphene which allows wrapping existing applications with minimal to no changes inside an enclave, requiring you to at best recompile existing applications with the provided tool set with support for common languages such as C, C++, Go, Java and Rust.
With its core being written in the memory-safe programming language Rust and a separated environment under which your application is running, it provides a safe yet powerful way to build your applications.
Running an Occlum app with Marblerun requires [minor changes]({{< ref "docs/building-services/occlum.md" >}}) to its manifest.
