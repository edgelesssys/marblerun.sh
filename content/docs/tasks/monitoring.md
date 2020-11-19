---
title: "Monitoring and logging"
date: 2020-11-14T16:28:16+05:30
draft: false
weight: 6
---

# Monitoring and logging

As of now, the monitoring capabilities of Marblerun are basic. For status information, the Coordinator provides the `/status` endpoint in the client API.
It returns the following information.

- **0 recovery mode**: Found a sealed state of an old seal key. Waiting for user input on [`/recover`]({{< ref "docs/features/recovery.md" >}}).
- **1 uninitialized**: Fresh start, initializing the Coordinator.
- **2 waiting for a manifest**: Waiting for user input on [`/manifest`]({{< ref "docs/tasks/set-manifest.md" >}})
- **3 accepting marbles**: Accepting Marbles through the [Marble API]({{< ref "docs/tasks/add-service.md" >}})

More details about the Coordinator can be retrieved through its log as follows.

```bash
kubectl -n marblerun logs -f marblerun-coordinator-5b9b4849c8-jbjwr
```

Going forward, plans are to support your favorite monitoring tool via logging through [Prometheus](https://prometheus.io/) and having a web dashboard with status and health information quickly accessible.
