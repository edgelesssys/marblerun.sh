---
title: "Recovery"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 3
---

# Recovery

Persistent storage for confidential applications in the cloud requires a bit of attention.
By design, SGX sealing keys are unique to a single CPU, which means using the default SGX sealing methods has some caveats.
For example, sealing data while running on one host could mean the data can't be unsealed when running on another host later on.

As described [in our secrets management chapter]({{< ref "docs/features/secrets-management.md" >}}), the Coordinator provides Marbles with virtual sealing keys making persistence straightforward for your applications.
Using virtual sealing keys, data can be unsealed independently of the physical host.

Still, the Coordinator itself must keep its state persistent somehow. When being pinned to a single host the default SGX sealing methods are used. However, when the Coordinator is moved to another physical host, a manual step is required to ensure the Coordinator's state can be recovered.
Therefore, the manifest allows for specifying a designated *Recovery Key*. The Recovery Key is a public RSA key. Upon startup, the Coordinator encrypts its symmetric state-encryption key with this public key. The holder of the corresponding private key can recover the Coordinator, as is described [in the recovery workflow]({{< ref "docs/workflows/recover-coordinator.md" >}}).

{{<note>}}
The owner of the Recovery Key can access the raw state of the Coordinator. In a future version, Marblerun will support splitting the Recovery Key between parties.
{{</note>}}
