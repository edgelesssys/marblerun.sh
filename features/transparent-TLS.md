# Transparent TLS

Authenticated and encrypted connections between services are essential for the security and verifiability of confidential applications. These properties are provided by mutual TLS authentication (mTLS). Normally, the applications inside the Marbles must support mTLS, be configured correctly, and be provisioned with the necessary secrets.

Transparent TLS (TTLS) can wrap any connection in TLS on the Marblerun layer. This means that Marblerun can add secure communication to your application even if it does not support the required TLS features. Just define the desired [connections in the Manifest](workflows/define-manifest.md#manifesttls).

TTLS is currently available with [EGo Marbles](building-services/ego.md). Other [runtimes](features/runtimes.md) will be supported in future.

## Authentication and credentials
By default the Marble's credentials are automatically configured. Connections between two Marbles are mutually authenticated.

You can use custom credentials defined in the Manifest's secrets. This can be useful when connecting from outside the cluster, to always serve the same certificate.
