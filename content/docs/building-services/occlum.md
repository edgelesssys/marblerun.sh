---
title: "Occlum"
draft: false
weight: 6
---

# Building a service: Occlum
Running an Occlum app with Marblerun requires some changes to its manifest.

## Requirements
Set up an environment to create Occlum images. For an easy start, we recommend that you use either [the official Occlum Docker image](https://hub.docker.com/r/occlum/occlum), or [use our provided Dockerfile](https://github.com/edgelesssys/marblerun/blob/master/samples/occlum-hello/Dockerfile). For a working DCAP remote attestation environment we recommend [our cloud deployment guide]({{< ref “/docs/deployments/cloud.md”>}}).

To build your service, you can start with Occlum's [Introduction](https://github.com/occlum/occlum#introduction) to get your application up and running, and then come back here to adapt it for use with Marblerun.

## Configuration
### Premain executable
Add our prebuilt [premain-occlum](https://github.com/edgelesssys/marblerun/releases/download/latest/premain-occlum) executable to your Occlum image, e.g., by copying it to `image/bin/premain-occlum`. By default, Occlum restricts executable files to the `/bin` directory. If you placed the `premain-occlum` binary to a different path, you need to adjust this setting accordingly.

Finally, define the original entry point for your Occlum instance as the first `Argv` parameter for your Marble in Marblerun's `manifest.json`. See [Defining a Manifest]({{< ref "docs/workflows/define-manifest.md" >}}) for more information on how to define the `Argv` parameters. This lets Marblerun launch your application after it succeeded in authenticating with the Coordinator and provides entrypoint pinning similar to the one offered in `Occlum.json`.

### Environment variables
The Marble needs to retrieve the Marblerun specific configuration parameters via environment variables, as [described under Step 3 in "Adding a service"]({{< ref "docs/workflows/add-service.md" >}}).

To pass environment variables to the enclave, Occlum requires them to be specified in the `env` section in `Occlum.json`.

You can provide default (hardcoded) values under `default`, and you may also define them additionally as `untrusted` in case you want to allow changes to the Marble configuration after build time.

For example, this configuration:
```json
"env": {
    "default": [
        "OCCLUM=yes",
        "EDG_MARBLE_COORDINATOR_ADDR=localhost:2001",
        "EDG_MARBLE_TYPE=hello",
        "EDG_MARBLE_UUID_FILE=uuid",
        "EDG_MARBLE_DNS_NAMES=localhost"
    ],
    "untrusted": [
        "EDG_MARBLE_COORDINATOR_ADDR",
        "EDG_MARBLE_TYPE",
        "EDG_MARBLE_UUID_FILE",
        "EDG_MARBLE_DNS_NAMES"
    ]
},
```

will allow you both to embed the expected default values during build time, but also let the user/host system change them during run time when a non-default Coordinator configuration is used.

### Resource limits
The premain process is written in Go. The enclave needs to have enough resources for the Go runtime, plus additional memory to launch your application.

We recommend starting with the following values which should work fine for light-wight to medium memory demanding applications:
```json
"user_space_size": "2048MB",
"default_mmap_size": "900MB"
"max_num_of_threads": 64
```

In case you are running into issues with memory demands, check out the [Resource Configuration Guide](https://github.com/occlum/occlum/blob/master/docs/resource_config_guide.md) provided by the Occlum team to debug and resolve issues related to resource limits.

## Troubleshooting
### failed to reserve page summary memory
If you receive the following fatal error during launch of your Occlum image:
```
fatal error: failed to reserve page summary memory
```

Make sure you allocated enough memory in `Occlum.json` [as described above](#resource-limits). The most important parameters are `user_space_size` and `default_mmap_size`.

### ERROR: The entrypoint does not seem to exist
If you receive this error message after the Marblerun premain executed:
```
ERROR: The entrypoint does not seem to exist: '/bin/your_application'
Please make sure that you define a valid entrypoint in your manifest (for example: /bin/hello_world).
panic: "invalid entrypoint definition in argv[0]"
```

or alternatively:

```
ERROR: Failed to spawn the target process.
Did you specify the correct target application in the Marblerun manifest as argv[0]?
Have you allocated enough memory?
panic: posix_spawn failed with error code -1
```

Make sure you specified the correct file name of your target application. For the latter error message, also make sure enough memory is allocated. To find out the specific reason for why this error is occurring, you can set the environment variable `OCCLUM_LOG_LEVEL=error` by appending it in front of your run command like this:

```sh
OCCLUM_LOG_LEVEL=error make run
```

or:
```sh
OCCLUM_LOG_LEVEL=error occlum run /bin/premain-occlum
```

Search for `SpawnMusl`. This entry will contain the error encountered when spawning your application from Marblerun's premain process.


### Error returned from the p_sgx_get_quote_config API

If Occlum crashes during the quote generation with the following error message:
```
[get_platform_quote_cert_data ../qe_logic.cpp:346] Error returned from the p_sgx_get_quote_config API. 0xe019
thread '<unnamed>' panicked at 'assertion failed: `(left == right)`
left: `SGX_QL_SUCCESS`,
right: `SGX_QL_NETWORK_ERROR`: fail to launch QE', src/util/sgx/dcap/quote_generator.rs:22:13
```

You might need to check the DCAP configuration on your system. Note that when using the Docker image, the local Intel DCAP configuration needs to be correctly set from **inside the container.**

If you use an Azure Confidential Computing machine, you can use our [provided Dockerfile](https://github.com/edgelesssys/marblerun/blob/master/samples/occlum-hello/Dockerfile) which patches the official Occlum image to use the Azure DCAP client, which handles the configuration automatically.

For other DCAP setups, please consult the documentation of your Intel Provisioning Certificate Caching Service (PCCS) service running locally or remotely.
