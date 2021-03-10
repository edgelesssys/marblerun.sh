---
title: "Building a Service: Graphene"
draft: false
weight: 6
---

# Building a Service: Graphene
Running a Graphene app with Marblerun requires some changes to its manifest. These are explained in the following. See also the [helloworld example]({{< ref "docs/examples/graphene.md" >}}).

### Entry Point and argv
We provide the `premain-graphene` executable with the [Marblerun Releases](https://github.com/edgelesssys/marblerun/releases). It will contact the Coordinator, set up the environment, and run the actual application. Therefore, you have to set it as the entry point of the Graphene project and place the actual entry point in argv0:
```toml
libos.entrypoint = "file:premain-graphene"
sgx.trusted_files.premain = "file:premain-graphene"
loader.argv0_override = "hello"
```

### Host Environment Variables
The premain needs access to some host [environment variables for configuration]({{< ref "docs/tasks/add-service.md#step-3-start-your-service" >}}):
```toml
loader.insecure__use_host_env = 1
```
The premain will remove all other variables before the actual application is launched, so this is secure.

### uuid file
The Marble must be able to store its uuid:
```toml
sgx.allowed_files.uuid = "file:uuid"
```

### Remote Attestation
The Marble will send an SGX quote to the Coordinator for remote attestation:
```toml
sgx.remote_attestation = 1
```

### Enclave Size and Threads
The premain process is written in Go. The enclave needs to have enough resources for the Go runtime:
```toml
sgx.enclave_size = "1024M"
sgx.thread_num = 16
```

## Secret Files
A Marble's secrets, e.g. a certificate and private key, can be provisioned as files. Ideally, these would be placed in the Marble's in-memory filesystem. Graphene does not support this yet, but you can fall back on *Graphene Protected Files* instead:
```toml
sgx.protected_files.cert = "file:server.crt"
sgx.protected_files.privkey = "file:server.key"
```
You can specify the files' content in the Marblerun Manifest:
```javascript
...
    "Parameters": {
        "Files": {
            "/dev/attestation/protected_files_key": "{{ hex .Marblerun.SealKey }}",
            "server.crt": "{{ pem .Secrets.server_cert.Cert }}",
            "server.key": "{{ pem .Secrets.server_cert.Private }}"
        }
    }
...
```
Note that Graphene requires to initialize the protected files key by writing it hex-encoded to the virtual `protected_files_key` device. This can be easily done through the above manifest configuration.

You can see this in action in the [nginx example]({{< ref "docs/examples/graphene.md" >}}).
