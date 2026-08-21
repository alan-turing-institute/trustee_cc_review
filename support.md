---
title: Confidential Computing Support
abstract: |
  An assessment of the current confidential computing offerings.
authors:
  - name: Jim Madge
    orcid: 0000-0001-6044-164X
license: CC-BY-4.0
keywords:
    - confidential computing
    - trusted execution environments
    - high performance computing
bibliography:
  - references.bib
---

## Current State of Confidential Computing

Confidential computing technology is still in a state of rapid development and prone to flux.
For example, AMD SEV technology has had a [number of extensions](https://www.amd.com/en/developer/sev.html) including SNP,
which includes important security improvements which could be considered essential for a TEE.
However, it has reached a state of maturity where the latest few generations of CPUs have support for robust and competent TREs.
This is mostly reserved for data-centre hardware, and not yet available on consumer devices.

Support utilising GPUs in TEEs is emerging, but less well developed that for CPUs.
Nvidia is currently the only vendor to offer GPU support in a TEE, and that is restricted to the lasted generation of Hopper devices.
This is discussed in [](#sec-ccs-vendor-nvidia-gpu)

(sec-ccs-cc-archetypes)=
### TEE Archectypes

Two key archetypes for TEEs have emerged.

(sec-ccs-cc-archetypes-vm)=
#### Secure VMs

VM-based TEEs are the most common approach, and appear to currently have the most focus from hardware manufacturers.
Examples include, [SEV-SNP](#sec-ccs-vendor-amd-cpu), [CCA](#sec-ccs-arm-cpu), and [TDX](#sec-ccs-vendor-intel-cpu).
They work by provisioning secure virtual machines on a hardened hypervisor.
The guest is secured through the encryption of all its memory using a key unique to that guest.
Therefore, if another process is able to read those memory addresses, they can only see the encrypted data.
A secure VM TEE implementation may further protect and isolate pages belonging to a TEE from inspection in more sophisticated attacks,
for example those targeting the hypervisor, exploiting out of date firmware, or requiring physical access.

The segregation of the secure guest from the host means that there is no need to trust the host OS and any software on the host.
The guest OS must be trusted and forms part of the TCB.
The root of trust remains the host CPU and hardware vendor.
An example block diagram showing the trusted and untrusted components of a VM-based TEE is shown in [](#fig-tcb-secure-vm).
In contrast to [the previous, similar example](#fig-tcb-and-rot) the guest OS is now a trusted component.

::::{figure}
:label:fig-tcb-secure-vm
:::{image} https://mermaid.ink/img/pako:eNp9UltvmzAY_SuW-7JJLIJwC95Tm6Svm7TuZWOaXDAXFTCyTZsuyn_fZ3NJoE0tJPyd73KO7XPECU8ZJvix4slT3CCU8KqrG4lcHRiU5IJ37V_FGFlr8FwzhJJn6oUKhlKqaA_lHZPqd4zNH337EeM_OsGaVP9gFOQe9nv06WF793lIXpIVXKolm9OHxWvLxHMpuehjrgomYJxumaRYKBXlMxNymA1tkOZyrFsq0iDkukaJTiqWGgCY65Y3rFHyHYmCX1OYlaLWImDguEW0SVFdJoLr-55EJW0HRdvvP-dqYLQWSkVqegXnCvEMGW0XVxY3SUWl3LEMVfSRVUBcVeTm3ixLKsGf2JeXMlUFsdvD16Fc335fPwLmrDMEBMwATWFO_QGF1x4GgNxEZk2Mk4H63Rw27O_gWsMlrDWMr9Or8Ha3997t9YNOvhzaRlzbdIn1Rl2A8DxLaHrQOa7Fnc3Ty3PDcH-3uS6vd-S5bcIng7_NGbNfwtjCuShTTABhFq6ZqKkO8VEbKcZQX4PfCGxTltGuUjGOmxO0tbT5xXk9dsJN5wUmGa0kRF0Ll8R2Jc0FrSdUgD-Z2HLgx2S9Cc0QTI74gIkTeCtvE0aOb7uRHax9C79i4gX2ynXt0NlEked4dnCy8D_Daq98J9Cf5wZ-5IaBf_oP-juVLQ?type=png
:::
The trusted and untrusted components of an example secure VM TEE.
% :::{mermaid}
% block
%   columns 3
%   block:group_tee:2
%     columns 2
%     software data
%     guest["guest OS"]
%   end
%   tee["TEE (TCB)"]
%   block:group_host:2
%     columns 1
%     hypervisor
%     other["host software, drivers"]
%     hostos["host OS"]
%   end
%   host["untrusted host components"]
%   block:group_rot:2
%     columns 1
%     firmware["firmware and microcode"]
%     cpu["CPU"]
%   end
%   rot["hardware root of trust (TCB)"]
% 
% classDef label fill:#FFFFFF,stroke-width:0px;
% class tee label
% class host label
% class rot label
% classDef group fill:#FFFFFF,stroke-width:4px,stroke:#999999;
% class group_tee group
% class group_host group
% class group_rot group
% classDef trusted fill:#4DAF4A,stroke-width:0px;
% class software trusted
% class data trusted
% class guest trusted
% class cpu trusted
% class firmware trusted
% classDef untrusted fill:#377EB8,stroke-width:0px;
% class hostos untrusted
% class hypervisor untrusted
% class other untrusted
% :::
::::

A secure VM TEE may provision a lower-trust portion of memory, which allows the transfer of data between the host and guest.

An advantage of the VM-based TEEs is their flexibility.
Little or no code changes will be required for software to run in such a TEE.
Building a secure application does not involve using confidential computing APIs (possibly with the exception of moving data between host and guest) a confidentiality is guaranteed by the hardware and VM.

(sec-ccs-cc-archetypes-enclave)=
#### Enclaves

Unlike [VM-based TEEs](#sec-ccs-cc-archetypes-vm), enclave TEEs run directly on the host.
Also in contrast, the OS running the trusted workload (in this case the host OS) is not trusted.
This is achieved by creating a secure enclave, a cryptographically segregated portion of the CPU and memory, where the trusted routines runs, before returning the result.
Only the functions which must be secure need to be run in the enclave.
Any other application code can run outside of the enclave.

The basic workflow for an enclave TEE is,

1. Untrusted code requests an enclave
1. Trusted hardware creates enclave
1. Untrusted code triggers attestation of enclave
1. Untrusted code decides whether to proceed
1. Untrusted code calls trusted routine
1. Trusted routine runs and returns result
1. Untrusted code processes result

[](#fig-tcb-enclave) is a block diagram showing the trusted and untrusted components of an enclave TEE.

::::{figure}
:label:fig-tcb-enclave
:::{image} https://mermaid.ink/img/pako:eNp9VNFumzAU_RXLfWURAVLAe2qb9nWTpr1smSoXTEAFG9kmzRbl33dtYgeyLESK7j0-99xjX8MBF6JkmOC3VhTvG45QIdqh4wrFJrEo2Uox9K-070lkQHQmuXxG1Iw5GCGoetVyUJqVPzf4FBkUNLpecMb1Bv9yfMZLF4IKFDBetHTHPMXIDfws6OOrkic5WAIq_LdNQXUjuJeb2q6F0m6Dbn_LMRW6ZhIkDAUpUekPKlmAStnsmFRezSwL5Xhfvs1dGHDm2LK8ZScztSTF_xxVjeyMCRB0IaK8RF1TSGFG6k0V_QCkp6_fPbIDQ0KO4CmZO4W2ZhNUllZXCqGRqJD1bZnjDyaj1JpVqKVvrAVLbUvuXuwTKC3FO_v00ZS6JmG__3yi2zFZvgNgznPAHssMAT8zwPS0B3SjZ9LvTwC5y-3jLfhbOkZz2Ha_ghsPV2CznSlsrLn5juaS9cNL8nDzQNwL4irdEozuEvLDvsDHMV6gxsz5uo124jR9fsxu2znX-Gg6HKH-xe0bMoVxgLeyKTEBhAW4Y7KjJsUHc8PgS1CzDi4pgbBkFR1auFkbfoSynvIfQnSuEg53W2NS0VZBNvQl1Wzd0K2knUcl7J7JJwH9MVkmWWxVMDngvcmTRbSMsjy8j1YB_o1JskoWeXIfL_NVFoZptjwG-I9tGS6yJM3jaBVlYRrnYRod_wL2yss9?type=png
:::
The trusted and untrusted components of an example enclave TEE.
% :::{mermaid}
% block
%   columns 3
%   block:group_app:2
%      columns 2
%       block:group_tee
%         app_trusted["trusted app component"]
%       end
%       tee["enclave"]
%     app_untrusted["untrusted app component"]
%   end
%   app["application"]
%     block:group_host:2
%     columns 1
%     other["host software, drivers"]
%     hostos["host OS"]
%   end
%   host["untrusted host components"]
%   block:group_rot:2
%     columns 1
%     firmware["firmware and microcode"]
%     cpu["CPU"]
%     vendor["CPU vendor"]
%   end
%   rot["hardware root of trust"]
% 
% classDef label fill:#FFFFFF,stroke-width:0px;
% class app label
% class tee label
% class host label
% class rot label
% classDef group fill:#FFFFFF,stroke-width:4px,stroke:#999999;
% class group_tee group
% class group_host group
% class group_rot group
% class group_app group
% classDef trusted fill:#4DAF4A,stroke-width:0px;
% class app_trusted trusted
% class cpu trusted
% class firmware trusted
% class vendor trusted
% classDef untrusted fill:#377EB8,stroke-width:0px;
% class app_untrusted untrusted
% class hostos untrusted
% class other untrusted
% :::
::::

Enclave TEEs have a smaller TCB compared to [VM-based TEEs](#sec-ccs-cc-archetypes-vm) as there is no guest OS, and only a small part of any application needs to be trusted.
This would make an in-depth security audit more feasible.
Developing for an enclave TEE will require calling CC APIs.
Translating an existing application would require splitting trusted routines and adding steps to create, attest and use enclaves to the untrusted part.

(sec-ccs-coco)=
### Confidential Containers

A more recent and promising approach to confidential computing is confidential containers [@ccc-terminology].
These are processes from OCI container images, launched by a container runtime, running in a TEE.
That way, the container process is protecting from the host.

[Confidential Containers](https://confidentialcontainers.org/) is a project of the [CNCF](https://www.cncf.io/) building open source tools to support running container applications in TEEs.
The project extends [Kata](https://katacontainers.io/), a container runtime which runs container processes in lightweight virtual machines, rather than conventional sandboxes using Linux namespaces and cgroups.
Confidential Containers [builds on Kata](https://confidentialcontainers.org/docs/architecture/design-overview/#kata-containers) by ensuring images and workload data are pulled by the confidential guest (not the host), handling attestation, and managing secure communication between the guest and external, trusted resources.
There is [existing support](https://confidentialcontainers.org/docs/overview/#what-hardware-does-confidential-containers-support) for a number of different [TEE implementations](#sec-ccs-vendor) and archetypes.
In addition to the container runtime, Confidential Containers maintains a set of trusted components to manage the lifecycle of ephemeral TEEs, including attestation and secret management.
These components run off of the confidential guest and are collectively called [Trustee](https://confidentialcontainers.org/docs/attestation/).

Confidential Containers uses one TEE per [pod](https://confidentialcontainers.org/docs/architecture/design-overview/#kata-containers).
This decision was made as a compromise between security and convenience.
The TCB is larger than if each container were run in its own TEE, but avoids the complex configuration required for containers in the same pod to share data.
An alternative approach, would be to build a Kubernetes cluster from CVM nodes.
This would be simpler to implement, however it greatly increases the size of the TCB as now all processes running on the kubernetes nodes, such as Kubelet and potentially etcd, would be within the TEE.

Each pod is therefore crytographically isolated from both the host and other pods in the K8s cluster.
Altogether, Confidential Containers represents a level 3 confidential computing implementation [@ccc-degrees] where attestation happens at a workload level and considers measurements of that workload.
The confidential guest is a minimal image only capable of producing measurements and executing approved processes.

Confidential Containers additionally provides [features](https://confidentialcontainers.org/docs/features/) to manage and interact with confidential containers.
These include being able to query attestation evidence in container processes, directing the confidential guests to pull from container image proxies, and providing container processes with access to encrypted secrets.
These tools can be used to build more complex workflows than simply running discrete confidential pods.

(sec-ccs-gpu)=
### GPUs and other devices

TEEs are CPU-based, with the important processes maintaining confidentiality and creating trust being handled by the {term}`secure processor`.
There is therefore a challenge integrating devices, such as GPUs,
with TEEs as there needs to be a mechanism for confidential data to be moved from the TEE to a trusted device without exposing it in plain text on the host.
Some TEE implementations allow for the creation of shared memory [@amd-memory-encryption].
These are pages which can be accessed by both the host and TEE, and are not encrypted by the TEEs memory encryption key.
This space can therefore be used to send messages between the host and TEE.

To communicate with a trusted device, shared pages can be used as a [bounce buffer](https://community.intel.com/t5/Blogs/Tech-Innovation/Artificial-Intelligence-AI/Confidential-AI-with-GPU-Acceleration-Bounce-Buffers-Offer-a/post/1740417).
Instructions from the TEE can be encrypted using a device public key and written to the shared memory, where the device can read before decrypting in it's own confidential computing zone.
This way, the communication between device and TEE are never exposed to the host in plain text.

An alternative approach is to allow DMA to the TEE from the device.
This approach is more performant as data does not need to pass through an intermediate buffer, reducing latency.
It is also potentially more secure as messages between TEE and device are never stored in memory pages available to the host OS or hypervisor.
However, to use a device in this way both the device and CPU must support a common method for this communication.
The [PCI-SIG](https://pcisig.com/) has produced [two specifications](https://pcisig.com/blog/ide-and-tdisp-overview-pcie%C2%AE-technology-security-features) for this purpose,

- [IDE](https://pcisig.com/PCI%20Express/ECN/Base/IntegrityandDataEncryption_A) provides a method to encrypt communication between TEE and devices over PCIe.
- The [TDISP](https://pcisig.com/PCI%20Express/ECN/Base/TEEDeviceInterfaceSecurityProtocol) protocol can be used to establish trust with, connect, and disconnect a device to a CVM.

As with the CPU, it is important to establish trust in devices connected to a TEE.
Before trust in a device has been established, a TEE should not use it.
With an untrusted connection established (either through a bounce buffer or TDISP), [evidence gathering](#sec-cc-attestation-evidence) similar to with a CPU can occur and claimed verified in an attestation report.

(sec-ccs-vendor)=
## Vendor Support

<!-- Table (or similar summary) of the vendors and processors considered -->

:::{table} Vendor TEE implementations
:label: tab-vendor
| Vendor   | Hardware   | Archetype                                   | Implementation                                  | Container Support[^cc]   |
| -------- | ---------- | ------------------------------------------- | -------------------------------------------     | ------------------------ |
| AMD      | CPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | SEV-SNP                                         | ✅ (with Kata)           |
| AMD      | GPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | TDISP                                           |                          |
| ARM      | CPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | CCA                                             | 🟠 (attestation only)    |
| Intel    | CPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | TDX                                             | ✅ (with Kata)           |
| Intel    | CPU        | [Enclave](#sec-ccs-cc-archetypes-enclave)   | SGX                                             | ✅                       |
| Nvidia   | GPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | bounce buffer or TDISP (from Blackwell onwards) |                          |
:::

[^cc]: Supported by the [Confidential Containers project](https://confidentialcontainers.org/docs/overview/)

### AMD

(sec-ccs-vendor-amd-cpu)=
#### CPU

##### Summary

SEV-SNP is AMD's TEE implementation.
It is a [secure VM](#sec-ccs-cc-archetypes-vm) TEE.
SEV was the original secure virtualisation implementation.
There have been a number of additions to these original features and support [varies across CPU generations](https://www.amd.com/en/developer/sev.html).
In particular, SNP adds important memory protections for hypervisor-based attacks.
SEV is a TEE, but care should be taken as without SEV-SNP, there are known vulnerabilities which could be exploited by those with access to the hypervisor or host system.

Communication between host and secure guests is supported in SEV.
Each guest is able to specify which pages of memory should be private (encrypted with the guest's unique key) and shared (encrypted with a hypervisor key).

Communication with devices using TDISP/IDE has been supported since 9005 (Turin) in a feature called Trusted IO.

##### Hardware Support

TEEs are a feature of the datacentre EPYC line of CPUs.
SEV is supported from EPYC 7001 (Naples) and SEV-SNP from 7003 (Milan).
The [AMD website](https://www.amd.com/en/developer/sev.html) has a table of features introduced across subsequent CPU generations.

(sec-ccs-vendor-amd-gpu)=
#### GPU

The [MI455X](https://www.amd.com/content/dam/amd/en/documents/products/accelerators/instinct/amd-instinct-mi455x_brochure.pdf) will be the first AMD GPU to support TEE integration.
Connection will be made through TDISP/IDE and compatible with Intel and AMD CPUs which also support those standards.
The [Helios](https://www.amd.com/content/dam/amd/en/documents/products/accelerators/instinct/amd-instinct-helios-blueprint-brochure.pdf) system will incorporate MI455Xs, Pensando NICs and AMD Venice CPUs.
Both the NICs and GPUs can be passed to CVMs.

#### Resources

- [AMD Infinity Guard](https://www.amd.com/en/products/processors/server/epyc/infinity-guard.html) security technologies
- [AMD SEV](https://www.amd.com/en/products/processors/server/epyc/infinity-guard.html) developer information
- [AMD confidential computing](https://www.amd.com/en/products/processors/server/epyc/confidential-computing.html)
- [SEV-SNP whitepaper](https://docs.amd.com/v/u/en-US/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more)
- [SEV-SNP attestation process](https://www.amd.com/content/dam/amd/en/documents/developer/lss-snp-attestation.pdf) presentation at Linux Security Summit 2022
- [Trusted I/O whitepaper](https://www.amd.com/content/dam/amd/en/documents/developer/sev-tio-whitepaper.pdf)

### ARM

(sec-ccs-arm-cpu)=
#### CPU

##### Summary

[CCA](https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture) is ARMs TEE implementation.
It is a secure VM approach, which depends on the strongly related RME technology, for managing isolated VMs (termed realms) on ARM platforms.
CCA is less mature than [SEV-SNP](#sec-ccs-vendor-amd-cpu) and [TDX](#sec-ccs-vendor-intel-cpu).
It is available on ARM datacentre CPUs but there is also an emphasis on making TEEs available on ARM-based mobile, automotive and embedded devices.

##### Hardware Support

CCA was introduced in with the RME instructions [ARMv9-A](https://www.arm.com/architecture/cpu/a-profile/armv9) in 2021.
The instructions may optionally be implemented on ARMv9-A CPUs
A number of Cotrex mobile and Neoverse datacentre CPUs have been developed with this or later, architectures.
The Neoverse V3 datacentre/cloud processors support CCA.
These have been manufactured into SoCs, AWS Graviton5, Azure Cobalt 200 and Nvidia Thor.

#### Resources

- [ARM CCA](https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture) feature summary
- [CCA on Fujitsu Monaka](https://developer.arm.com/community/arm-community-blogs/b/servers-and-cloud-computing-blog/posts/how-fujitsu-implemented-confidential-computing-on-fujitsu-monaka-with-arm-cca) upcoming CPU blog post
- [RME developer guide](https://developer.arm.com/documentation/den0126/latest)

### Intel

(sec-ccs-vendor-intel-cpu)=
#### CPU

##### Summary
Intel has two TEE implementations, SGX and TDX
TEE technology on Intel CPUs is implemented as part of the IME.
This, is an off-CPU subsystem built into the chipsets of Intel motherboards.

:::{warning}
As part of the IME is implemented in the chipset,
it may be possible for a TEE-capable CPU to be socketed in a motherboard with an incompatible chipset or one with out of date firmware.
This makes the TCB larger, or at least spreads its components across more places.
:::

The older of the two implementations is SGX.
[Intel SGX](https://www.intel.com/content/www/us/en/products/docs/accelerator-engines/software-guard-extensions.html) is an [enclave](#sec-ccs-cc-archetypes-enclave) and hence allows TEEs to run directly on a compatible host system.
SGX also refers to the instruction set extension for x86 that implement the TEE.
As developing for enclave TEEs requires significant code changes, Intel produces an SDK for SGX applications.

[Intel TDX](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html) is a newer system for confidential computing than SGX.
Unlike SGX, TDX is a [secure VM](#sec-ccs-cc-archetypes-vm) TEE.
The confidential VMs are referred to as trust domains.
Communication between the host and TEE is possible through a shared portion of memory, the Shared Extended Page Table.

TDX TEEs support integrating devices through TDISP/IDE in a feature called [TDX Connect](https://www.intel.com/content/www/us/en/content-details/862706/intel-tdx-connect-architecture-specification.html)

##### Hardware Support

SGX was previously available on both consumer and datacentre processors, with support going back to the Skylake family in 2015.
Support has been removed in recent consumer devices, as Intel focuses more on developing TDX.
Support has remained for the latest generations of Xeron processors, with Intel indicating that they are committed to [continued support](https://community.intel.com/t5/Blogs/Products-and-Solutions/Security/Rising-to-the-Challenge-Data-Security-with-Intel-Confidential/post/1353141) on Xeon.

TDX is supported on the 5th and 6th Generation Xeon (Emerald Rapids and Granite Rapids respectively) CPUs.
Some cloud providers have special SKUs of 4th Generation Xeon (Sapphire Rapids) which are TDX-compatible, however these are not generally available.

#### Resources

- [SGX](https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html)
- [SGX Developers Guide](https://download.01.org/intel-sgx/latest/linux-latest/docs/Intel_SGX_Developer_Guide.pdf)
- [Intel SGX Use and Development Flow](https://www.intel.com/content/www/us/en/content-details/671417/intel-software-guard-extensions-and-the-development-flow-infographic.html)
- [TDX](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html)
- [Which Intel TEE is right for you](https://www.intel.com/content/www/us/en/content-details/818845/which-intel-trusted-execution-environment-is-right-for-you.html) graphic
- [Intel Trust Authority](https://www.intel.com/content/www/us/en/security/trust-authority.html) attestation service
- [TDX Connect](https://www.intel.com/content/www/us/en/content-details/862706/intel-tdx-connect-architecture-specification.html) specification

#### GPU

Intel GPUs currently have no support for TEEs.

### Nvidia

(sec-ccs-vendor-nvidia-gpu)=
#### GPU

##### Summary

For its recent GPU architectures, Nvidia has developed technology to allow its accelerators to be attested and used in secure guests [@nvidia-secure-ai].
Integration of Nvidia GPUs is supported for SEV-SNP, and TDX.

##### Hardware Support

Confidential computing is supported on the Hopper, Blackwell and Rubin architectures of datacentre GPUs.
When provisioned on a TEE capable host from a compatible vendor, a GPU from these families may be used by a CVM.
Bounce buffers can be used with all families but DMA with TDISP/IDE is only supported from Blackwell onwards [@nvidia-secure-ai].
Single, or multiple GPUs can be connected to a CVM, however, with Hopper this is limited to specific HGX products.

:::{important} Grace Hopper
Nvidia Grace Hopper superchips do not support confidential computing.
Despite packaging a Hopper GPU with an ARMv9 CPU, the CPU model (Neoverse V2) does not include RME instructions.
:::

The announced [Vera Rubin superchip](https://nvidianews.nvidia.com/news/rubin-platform-ai-supercomputer) will support confidential computing.
It is comprised of Rubin GPUs and Vera CPUs (with custom ARMv9-A cores) on a tightly integrated datacentre rack.

#### Resources

- [Nvidia confidential computing](https://www.nvidia.com/en-us/data-center/solutions/confidential-computing/)
- [H100 confidential computing](https://developer.nvidia.com/blog/confidential-computing-on-h100-gpus-for-secure-and-trustworthy-ai/) blog post
- [Vera Rubin](https://developer.nvidia.com/blog/inside-the-nvidia-rubin-platform-six-new-chips-one-ai-supercomputer/) blog post
