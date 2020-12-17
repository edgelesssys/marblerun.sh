---
title: "Changelog"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 10
---

# Changelog

## Version 0.2.0

* Switch from simple `$$<key-name>` placeholders to more versatile Go Templates for secrets injection in the Manifest.
* Add `Secrets` section to Manifest for the definition of custom keys and certificates.
* Add recovery API
* Add a Prometheus metrics endpoint
* Use ertgolib's new feature for retrieving Marblerun TLS credentials transparently
