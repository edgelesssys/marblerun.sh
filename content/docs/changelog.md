---
title: "Changelog"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 10
---

# Changelog

## Version 0.3.0

* Add support for privileged Client API endpoints
* Allow increasing the minimum required `SecurityVersion` for packages after a manifest has been set initially (also referred to as 'Manifest Update')
* Add an intermediate certificate to the Marblerun certificate chain. Keep the root certificate permanent and change the intermediate with each manifest update. Return the whole cert-chain on the client API. Clients can pin the intermediate for automatically catching a manifest update. Alternatively, they can pin the root certificate that will be valid for the lifetime of the service mesh.
* Add a command-line interface (`CLI`) for administrating Marblerun
* Add integration for `Graphene`-based Marbles
* Add a Kubernetes mutating admission webhook as `marble-injector` service for automatically providing Pods with their Marble configuration
* Refactor the client API as an HTTP-REST interface with responses following the JSend style
* Use EGo for the Golang helloworld samples
* Move libertmeshpremain into EdgelessRT

## Version 0.2.0

* Switch from simple `$$<key-name>` placeholders to more versatile Go Templates for secrets injection in the Manifest.
* Add `Secrets` section to Manifest for the definition of custom keys and certificates.
* Add recovery API.
* Add a Prometheus metrics endpoint.
* Use ertgolib's new feature for retrieving Marblerun TLS credentials transparently.