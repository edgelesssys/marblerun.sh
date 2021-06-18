---
title: "Changelog"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 10
---

# Changelog

## Version 0.3.2
* CLI:
  * Add auto-complete support
  * Add `sgxsdk-package-info`: Prints the package signature properties of an SGX SDK binary
  * Kubernetes SGX-device-plugin refactoring:
    * Don't install own device-plugin with Marblerun
    * Auto-detect installed device-plugin
    * Add support for Intel's device-plugin
* Marbles:
  * EGo:
    * Let EGo handle filesystem mounts
  * Occlum:
    * Add support for Occlum workloads
  * Graphene:
    * Use execve instead of posix_spawn due to performance
    * Remove obsolete preload option for premain
* Cleanup and improve BUILD.md
* Cleanup and improve examples
* Miscellaneous bug fixes
  * Fix several issues with CLI when Marblerun runs in standalone mode
  * Fallback to latest era-config if CLI can't find version running

## Version 0.3.1

* Add CLI features:
    * `check`: Check the status of Marbleruns control plane
    * `graphene-prepare` Modifies a Graphene manifest for use with Marblerun
    * `precheck`: Check if your Kubernetes cluster supports SGX
    * `uninstall` Removes Marblerun from a Kubernetes cluster
    * `version`: Display version of this CLI and (if running) the Marblerun coordinator
    * Add support of the YAML format for manifests in the CLI
    * Print failure reason when setting a manifest fails
* Miscellaneous bug fixes
    * Fix a crash where an empty quote could cause the coordinator to crash
    * Fix a potential segfault on Marble activation
    * Fix a bug in the marble-injector Kubernetes MutatingAdmissionController deployment. Not setting the `CABundle` for Kubernetes version 1.19 and higher resulted in a failed authentication of the webhook in the Kubernetes API server. Ultimately, this leads to failing deployments of Marble pods

## Version 0.3.0

* Add support for privileged Client API endpoints
* Allow increasing the minimum required `SecurityVersion` for packages after a manifest has been set initially (also referred to as 'Manifest Update')
* Add an intermediate certificate to the Marblerun certificate chain. Keep the root certificate permanent and change the intermediate with each manifest update. Return the whole cert chain on the client API. Clients can pin the intermediate CA for automatically catching a manifest update. Alternatively, they can pin the root certificate that will be valid for the lifetime of  the deployment.
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
