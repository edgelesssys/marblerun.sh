---
title: "On-premises"
draft: false
weight: 3
---

# On-premises Marblerun deployment

This guide should give you all the information for setting up a Marblerun deployment on your local machine or on-premises cluster.

Marblerun without Intel SGX can be deployed on arbitrary bare-metal machines or VMs. Please see
our [quickstart]({{< ref "docs/getting-started/quickstart.md" >}}) guide for the details.


## Prerequisites
### Hardware and firmware

**CPU**

To deploy Marblerun with Intel SGX, the machine or VM has to support Intel SGX.
Particularly, Marblerun requires support for the SGX Datacenter Attestation Primitives (DCAP).
[You can verify if your CPU supports DCAP](https://www.intel.com/content/www/us/en/support/articles/000057420/software/intel-security-products.html):

If a processor supports Intel SGX and FLC, it supports DCAP.

There are two options to determine if your system's processor supports FLC:

**Option 1**: On a Linux system, execute [cpuid](http://manpages.ubuntu.com/manpages/cosmic/man1/cpuid.1.html) in a terminal:

1. Open a terminal and run: `$ cpuid | grep -i sgx`
2. Look for output: `SGX_LC: SGX launch config supported = true`

**Option 2**: Use [test-sgx.c](https://github.com/ayeks/SGX-hardware#test-sgx):
1. Go to the [SGX Hardware Github](https://github.com/ayeks/SGX-hardware) and download the file test-sgx.c or clone the [repository](https://github.com/ayeks/SGX-hardware)
2. Compile and run test-sgx.c according to these [instructions](https://github.com/ayeks/SGX-hardware#test-sgx):
    ```bash
    $ gcc test-sgx.c -o test-sgx
    $ ./test-sgx
    ```
3. Look for output: `sgx launch control: 1`

For more information read the [Properly Detecting Intel Software Guard Extensions (Intel® SGX) in Your Applications](https://software.intel.com/content/www/us/en/develop/articles/properly-detecting-intel-software-guard-extensions-in-your-applications.html) article.

You can find more information on DCAP in [An update on 3rd Party Attestation.](https://software.intel.com/content/www/us/en/develop/blogs/an-update-on-3rd-party-attestation.html)

**BIOS**

The system BIOS must support Intel SGX control.

BIOS support is required for Intel SGX to provide the capability to enable and configure the Intel SGX feature in the system. The system owner must opt into Intel SGX by enabling it via the BIOS. This requires a BIOS from the OEM that explicitly supports Intel SGX. The support provided by the BIOS can vary from OEM to OEM and even across an OEM’s product lines.

Given the SGX feature is relatively new, not all BIOS support the control of the SGX feature even if a system is built with an SGX capable Intel processor. You would need to boot into BIOS to see if your BIOS supports SGX control.

Intel SGX must be enabled in the BIOS.

Currently, most of the SGX capable systems have SGX disabled by default in the BIOS. This default setting might change but for now, you need to manually enable it if it's not already enabled.

**Updates**

As with any modern technology, Intel SGX has been affected by security vulnerabilities. Intel addresses these vulnerabilities by updating the microcode of CPUs, changing the hardware of new CPUs, and updating the system software. Each microcode update that patches an SGX vulnerability requires a BIOS update. During remote attestation, it is checked that the microcode of the CPU which is deployed by the BIOS is up to date. The microcode and platform enclaves are commonly called the platform `Trusted Computing Base (TCB)`.

If your BIOS/firmware is outdated, you will see errors as `Platform TCB (2) is not up-to-date (oe_result_t=OE_TCB_LEVEL_INVALID)` during remote attestation procedures.

**Hypervisor**

If you are using VMs for your Marblerun deployment, you need to make sure your hypervisor has SGX enabled.
Most of the popular hypervisors support SGX:

* [QEMU/KVM](https://software.intel.com/content/www/us/en/develop/articles/virtualizing-intel-software-guard-extensions-with-kvm-and-qemu.html)
* [XEN](https://wiki.xenproject.org/wiki/Xen_and_Intel_Hardware-Assisted_Virtualization_Security)
* Hyper-V: Hyper-V will only expose SGX to Gen 2 VMs
* [VMWare vSphere](https://blogs.vmware.com/vsphere/2020/04/vsphere-7-vsgx-secure-enclaves.html)
* [ACRN](https://projectacrn.github.io/latest/tutorials/sgx_virtualization.html)


**Driver**

You need to [install the SGX Driver for systems supporting the Launch Control Configuration](https://download.01.org/intel-sgx/latest/dcap-latest/linux/docs/Intel_SGX_DCAP_Linux_SW_Installation_Guide.pdf).
Azure provides the [instructions on how to install this driver](https://docs.microsoft.com/en-us/azure/confidential-computing/quick-create-portal#2-install-the-intel-sgx-dcap-driver) that you can use for your on-premises machine.
### SGX Datacenter Attestation Primitives (DCAP)

DCAP is the new attestation mechanism for SGX [replacing EPID]((https://software.intel.com/content/www/us/en/develop/blogs/an-update-on-3rd-party-attestation.html)).
You can find an overview of DCAP in the [official Intel docs](https://download.01.org/intel-sgx/dcap-1.1/linux/docs/Intel_SGX_DCAP_ECDSA_Orientation.pdf).
Marblerun only supports DCAP and requires DCAP libraries installed and configured on your system.

From the perspective of Marblerun and your workloads DCAP is accessed with a [Quote Generation Library (QGL)](https://github.com/intel/SGXDataCenterAttestationPrimitives/blob/master/QuoteGeneration/README.md) and a [Quote Verification Library (QVL)](https://github.com/intel/SGXDataCenterAttestationPrimitives/blob/master/QuoteVerification/README.md) for generating and verifying quotes respectively.

You currently have two options regarding DCAP for your on-premises machines and clusters:

1. Use a public DCAP service by configuring your QGL and QVL to point to the public endpoints. Currently, only Azure provides such a service that can be used by installing the [Azure-DCAP-Client](https://github.com/microsoft/Azure-DCAP-Client) package on your system:

    ```bash
    echo 'deb [arch=amd64] https://download.01.org/intel-sgx/sgx_repo/ubuntu bionic main' | sudo tee /etc/apt/sources.list.d/intel-sgx.list
    wget -qO - https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key | sudo apt-key add -
    echo "deb [arch=amd64] https://packages.microsoft.com/ubuntu/18.04/prod bionic main" | sudo tee /etc/apt/sources.list.d/msprod.list
    wget -qO - https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
    sudo apt update && sudo apt install -y libsgx-dcap-ql libsgx-dcap-ql-dev az-dcap-client
    ```

2. Run your own PCCS and expose it to your machine or cluster. See the following section for more information on PCCS and how to set up your own service.

**Provisioning Certificate Caching Service (PCCS)**

Many CSPs and data centers prevent their platforms from accessing the Internet directly. In
addition, they do not rely on an externally hosted service to perform runtime operations (for
example, the Intel SGX Remote Attestation).
The Caching Service for Intel SGX PCS allows a
CSP or a datacenter to cache PCK Certificates, PCK Certificate Revocation Lists (CRL), TCBInfo,
and QE Identity structures for all platforms in its cloud or data center. The PCK Certificates, PCK
CRLs, the TCBInfo, and QE Identity structures are all signed and published by Intel.

Currently there exist two popular implementations with their respective libraries:

* [Intel Demo PCCS](https://github.com/intel/SGXDataCenterAttestationPrimitives/blob/master/QuoteGeneration/pccs/README.md)
    * Install and run your own PCCS:
      * Option 1: [Use the Intel's demo PCCS](https://github.com/intel/SGXDataCenterAttestationPrimitives/tree/master/QuoteGeneration/pccs#how-to-install)
      * Option 2: [Write your own PCCS](https://download.01.org/intel-sgx/latest/dcap-latest/linux/docs/SGX_DCAP_Caching_Service_Design_Guide.pdf)
    * On your machines you need to install:
        * The DCAP client libraries
          * Build and install them from [source](https://github.com/intel/SGXDataCenterAttestationPrimitives#build-and-install-the-intelr-sgx-dcap-quote-generation-library)
          * Follow the [Installation Guide](https://download.01.org/intel-sgx/latest/dcap-latest/linux/docs/Intel_SGX_DCAP_Linux_SW_Installation_Guide.pdf) for the official packages
      * Install a [configuration that points to your PCCS](https://github.com/intel/SGXDataCenterAttestationPrimitives/blob/master/QuoteGeneration/qpl/README.md#configuration)

The PCCS is a cache, so you need to make sure it stays up to date. In case your cache is outdated, you might see error messages as `coordinator-enclave.signed:mbedtls_x509_crt_verify failed with The CRL is expired (flags=0x20) (oe_result_t=OE_VERIFY_CRL_EXPIRED)`.

You can inspect the Intel Root CA CRL of your PCCS:

```bash
curl --insecure --request GET --url https://<YOUR_PCCS_DOMAIN>:<YOUR_PCCS_PORT>/sgx/certification/v2/rootcacrl > rootca.crl
openssl crl -inform PEM -text -noout -in rootca.crl
```


You can refresh all SGX collaterals for your PCCS:

```bash
curl --insecure --request GET -H "admin-token: <my password>" --url https://<YOUR_PCCS_DOMAIN>:<YOUR_PCCS_PORT>/sgx/certification/v2/refresh
```

If refreshing CRL fails, you can manually delete the `pckcache.db` database (default location `/opt/intel/sgx-dcap-pccs/pckcache.db`) and restart your PCCS.

{{<note>}}
Currently, the Marblerun Coordinator and EdgelessRT docker images are configured to always use the public Azure PCCS.
In the future, we will make this configurable, so you can define your own PCCS for our images.
{{</note>}}

## Deploy Marblerun

You have made sure your hardware supports SGX, updated all firmware, installed the SGX driver, and configured DCAP on all your machines and VMs?
Great! Now it is time to install Marblerun and get going.

You can either follow our guide and [use Marblerun in standalone mode]({{< ref "docs/deployment/standalone.md" >}}) on your local machine.

Or you follow our instructions to [install Marblerun in your on-premises Kubernetes cluster]({{< ref "docs/deployment/kubernetes.md" >}}).
