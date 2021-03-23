---
title: "Recovery"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 2
---

# Recovery

Persistent storage for confidential applications in the cloud requires a bit of attention.
By design, SGX sealing keys are unique to a single CPU, which means using the default SGX sealing methods has some caveats.
For example, sealing data while running on one host could mean the data can't be unsealed when running on another host later on.

As described [in our secrets management chapter]({{< ref "docs/features/secrets-management.md" >}}), the Coordinator provides Marbles with virtual sealing keys making persistence straightforward for your applications.
Using virtual sealing keys, data can be unsealed independently of the host a Marble is running on.

Still, the Coordinator itself must keep its state persistent somehow. When being pinned to a single host the default SGX sealing methods are used. However, when the Coordinator is moved to a another physical host, a manual step is required to ensure the Coordinator's state can be recovered.
Therefore, the Manifest allows for specifying a special *Recovery Key*. The Recovery Key is a public RSA key. Upon startup, the Coordinator encrypts its own symmetric state-encryption key for this public key. The holder of the corresponding private key can use this key to recover the Coordinator, as is described [in our recovery chapter]({{< ref "docs/tasks/recover-coordinator.md" >}}).

Note that the holder of the private key is also able to manipulate the state of the Coordinator. In a future version, Marblerun will support splitting the Recovery Key between a group of individuals who all must come together to recover or alter the Coordinator.
