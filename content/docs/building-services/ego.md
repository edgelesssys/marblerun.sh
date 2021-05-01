---
title: "Building a service: EGo"
draft: false
weight: 5
---

# Building a service: EGo
To get your Go service ready for Marblerun, you possibly need to adapt its code slightly and you need to rebuild it. Details are given in the following.

{{<note>}}
Future versions of EGo will support building Marblerun services with zero changes.
{{</note>}}

## Make your service use the provided TLS credentials

If your service already uses TLS and gets the credentials from, e.g., a file, you just need to [adapt the Manifest]({{< ref "docs/tasks/add-service.md#make-your-service-use-the-provided-tls-credentials" >}}). Otherwise, you need to make small code changes.

We provide a convenience package called [github.com/edgelesssys/ego/marble](https://pkg.go.dev/github.com/edgelesssys/ego/marble#GetTLSConfig). With it, a service can automatically get and use its Marblerun TLS credentials. The following gives an example.
```Go
    serverCfg, err := marble.GetTLSConfig(false)
    if err != nil {
        log.Fatalf("Failed to retrieve server TLS config")
    }
    // use serverCfg, e.g., to create an HTTPS server
```

## Re-compile/build your service for Marblerun

Finally, you need to re-build your service for the enclave environment with `ego-go` and sign it with `ego sign`. Please follow the build instructions for Go provided in our [Go sample](https://github.com/edgelesssys/marblerun/blob/master/samples/helloworld).
