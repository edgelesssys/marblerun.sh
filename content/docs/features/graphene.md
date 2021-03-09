---
title: "Graphene integration"
draft: false
weight: 4
---

# Graphene integration

Marblerun has support for the [Graphene library OS](https://grapheneproject.io/).
Graphene is a popular choice for wrapping unmodified applications into enclaves.
This approach commonly known as "lift and shift" facilitates the process of bringing existing applications into the confidential space.
Graphene further adds support for dynamically linked libraries and multi-process applications in SGX.

In Marblerun. In essence, you have to add the `premain` process to the Graphene manifest. `premain` will contact the Coordinator, set up the environment, and run the actual application.
See the commented [hello.manifest.template](https://github.com/edgelesssys/marblerun/blob/master/samples/graphene-hello/hello.manifest.template) for details.

We provide two examples for Graphene-based Marbles:

* A [helloworld](https://github.com/edgelesssys/marblerun/tree/master/samples/graphene-hello) example to get you started. :nerd_face:
* An [NGINX webserver](https://github.com/edgelesssys/marblerun/tree/master/samples/graphene-nginx) for an example of converting an existing Graphene application to a Marble. :rocket: