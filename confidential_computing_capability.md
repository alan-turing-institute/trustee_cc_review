---
title: Confidential Computing Capability
abstract: |
  An assessment of the current confidential computing offerings and their availablity to the UK research community.
abbreviations:
    CCA: Confidential Compute Architecture
    CNCF: Cloud Native Computing Foundation
    CPU: Central Processing Unit
    EAT: Entity Attestation Token
    GPU: Graphics Processing Unit
    IETF: Internet Engineering Task Force
    IME: Intel Management Environment
    JWT: JSON Web Token
    OS: Operating System
    PROM: programmable read-only memory
    RFC: request for comments
    RME: Realm Management Extensions
    SEV: Secure Encrypted Virtualisation
    SNP: Secure Nested Paging
    SoC: System on a Chip
    TCB: Trusted Compute Base
    TEE: Trusted Execution Environment
    VM: Virtual Machine
authors:
  - name: Jim Madge
    orcid: 0000-0001-6044-164X
license: CC-BY-4.0
keywords:
    - confidential computing
    - trusted research
    - high performance computing
bibliography:
  - references.bib
---

## Confidential Computing

Confidential computing describes a set of technologies that protect data while it is in use.
This is distinct from protecting data at rest, written to a storage device, or in transit, while being sent over a network.
We could consider both of those as solved problems, with public/private key encryption, secure protocols and modern filesystems.
Confidential computing is achieved by creating a hardware-based, attestable TEE.
Attestation, proves that a TEE is correctly configured and has not been tampered with.
Data in use, and code being executing in a TEE is encrypted and may not be read or modified by other processes on the same computer.

### Trusted Execution Environments

A TEE is a reserved portion of hardware (CPU, memory and possibly GPU) that is cryptographically segregated from the rest of the system (the host, and any other TEEs).
The contents of a TEE are unable to be read, or modified by any processes outside of the TEE, including those running on the same host.
A TEE can prove the integrity of its TCB though an [attestation](#sec-cc-attestation) process, so its users can be informed and access whether to trust it.

In any computer system the TCB defines of all of the components (hardware, software and firmware) that play a role in providing the required security.
The term derives from the fact that we must trust these components perform their intended role as we expect,
as a flaw on any one member of the TCB could mean the security of the entire system is compromised.
A general principle is to minimise the size of the TCB, in other words, reducing the number of critical component for security.
This simplifies management and monitoring of the TCB and presents fewer routes for a malicious attack.

The TCB of a TEE can vary based on the [design of the TEE](#sec-ccs-cc-archetypes).
However, it will always exclude the host OS and any hypervisor as the TEE is cryptographically segregated from these by definition.
Although the host CPU, and its firmware and microcode, are critical to the security of a TEE, they are not considered part of the TCB.
Instead, they may be considered the "hardware root of trust" as CPU routines,
and the ability of a hardware manufacturer to validate genuine hardware,
play a critical role in the [attestation](#sec-cc-attestation) process.
The CPU and CPU manufacturer are therefore the root of the chain of trust for the entire TEE and ultimately you must trust these organisation and their products.
An overview of this is shown in [](#fig-tcb-and-rot)

::::{figure}
:label: fig-tcb-and-rot
:::{mermaid}
block
  columns 3
  block:group_tee:2
    columns 2
    software
    data
  end
  tee["TEE and TCB"]
  block:group_host:2
    columns 1
    hypervisor
    other["host software, drivers"]
    hostos["host OS"]
  end
  host["untrusted host components"]
  block:group_rot:2
    columns 1
    firmware["firmware and microcode"]
    cpu["CPU"]
    vendor["CPU vendor"]
  end
  rot["hardware root of trust"]

classDef label fill:#FFFFFF,stroke-width:0px;
class tee label
class host label
class rot label
classDef group fill:#FFFFFF,stroke-width:4px,stroke:#999999;
class group_tee group
class group_host group
class group_rot group
classDef trusted fill:#4DAF4A,stroke-width:0px;
class software trusted
class data trusted
class cpu trusted
class firmware trusted
class vendor trusted
classDef untrusted fill:#377EB8,stroke-width:0px;
class hostos untrusted
class hypervisor untrusted
class other untrusted
:::
A block diagram showing an example of a TEE implementation, showing the TCB, the untrusted host resources, and the hardware root of trust.
::::

(sec-cc-attestation)=
### Attestation

Attestation is the process by which a TEE proves that it is secure.
This is how trust is established when interacting with a TEE.
Attestation verifies that,

- The TEE has been configured correctly
- The TEE is running on authentic hardware

As part of the attestation process, the authenticity of hardware can be verified by a private/public key cryptography challenge.
Unique, unpredictable, random private keys for each CPU are generated at the time of manufacture.
They are written to the hardware in one-time PROM, and so are immutable.
The manufacturer keeps set of corresponding public keys to use in identity verification challenges.

In some cases, it may also verify that some software component of the TEE is in a known state, _i.e._ it has not been modified or tampered with.

#### Attestation Evidence

In order to attest the legitimacy of a TEE, evidence about that TEEs state must be assessed.
The evidence consists of measurements about the TEE.
The measurements vary based on the TEE implementation and may include,

- Information about the host and its hardware, including firmware and microcode
- In the case of a VM-based TEE, information about the guest and hypervisor
- Information about the software in a TEE

Measurements are collected as an evidence object, such as JWT [@rfc7519] or EAT [@rfc9711].
Evidence is cryptographic data, collected by hardware-based routines[^firmware-microcode], making it unfeasible to tamper with the evidence collection process or to mimic a legitimate system through virtualisation.
It is therefore reliable claim about the state of a TEE, which can be compared with a policy (set of requirements) when deciding whether to trust or interact with the TEE.

[^firmware-microcode]: In principle this also includes firmware and microcode, which may be modified after manufacture. The versions and integrity of firmware and microcode should therefore be verified as part of attestation.

#### Remote Attestation

The specific attention process we will discuss here is remote attestation.
In brief, remote attestation is a process that uses a third party (_i.e._ neither the TEE, nor the part requesting verification of the TEE) to assess the TEE.
This is defined formally in IETF's  request for comments 9934 @rfc9334
The RFC sets out a number of roles, some of which are paraphrased here.

:::{glossary}
attester
: The party (usually a machine) that that is being tested to prove it is trustworthy.
The attester provides evidence to be assessed to the {term}`verifier`.

verifier
: The party (or service) which assesses the evidence produced by the attester.
The verifier may consult {term}`endorsers <endorser>`, delegating the assessment of pieces of evidence.
The verifier does not itself decide whether the attester is trustworthy or not, but instead produces a report.

endorser
: A party (or service) that is consulted by the {term}`verifier` to assess pieces of evidence from the {term}`attester`.
Hardware manufacturers are an important class of endorser, holding secrets needed to verify the unique keys of genuine hardware.

relying party
: The party (perhaps a piece of software) which relies on the {term}`verifiers <verifier>` report in order to make a decision, such as whether to use a TEE for a workflow.

relying party owner
: The party (for example an organisation, or system administrator) which has the authority to control how the {term}`relying party` responds to an attestation report.
:::

:::{important} Attestation result
It is important to understand that the {term}`verifier` _does not_ make a decision on behalf of the {term}`relying party` as to whether they should trust a TEE.
It is always the responsibility of the {term}`relying party` to inspect the attestation report and decide what to do following the {term}`relying party owner's <relying party owner>` policy.
:::

:::{note} Trust
The roles and remote attestation process illustrates where trust lies when using a TEE.
The {term}`attester` does not validate itself or make a statement about its security, it produces evidence.
The {term}`verifier` processes evidence, possibly drawing from {term}`endorsers <endorser>` and must be trusted to correctly arrive at its conclusions.
As an extension, any {term}`endorsers <endorser>` must also be trusted to provide correct information to the {term}`verifier`.
:::

The basic workflow of attestation is shown in [](#fig-attestation).

::::{figure}
:label: fig-attestation
:::{mermaid}
flowchart LR
  Attestor
  Verifier
  Endorser
  rp[Relying Party]
  rpo[Relying Party Owner]
  Attestor --> |Evidence| Verifier
  Endorser --> |Endorsements| Verifier
  Verifier --> |Attestation result| rp
  rpo --> |Policy| rp
:::
The flow of information for a TEE attestation.
::::

The actual procedure to achieve remote attestation may be implemented in different ways.
@rfc9334 describes two possible patterns.

In the [passport model](https://www.ietf.org/rfc/rfc9334.html#name-passport-model) the {term}`attester` requests an attestation report from the {term}`verifier`, who then returns the attestation results directly back to the {term}`attester`.
The {term}`attester` shares the report with the {term}`relying party`.
This means the {term}`relying party` does not need to interact directly with the {term}`verifier` and an {term}`attester` may cache a result to reuse.
The model is named from the similarity of this process with a person issued with a passport that they can present to prove their identity.

In contrast, in the [background check model](https://www.ietf.org/rfc/rfc9334.html#name-background-check-model) the {term}`attester` provides evidence directly to the {term}`relying party`.
It is then the {term}`relying party` who sends the evidence to the {term}`verifier` and requests an attestation result.
Unlike the passport model, here the {term}`relying party` interacts with both the {term}`attester` and {term}`verifier`, and will possess both the attestation report and the {term}`attester's <attester>` evidence.

Neither of these possibilities are enforced or recommended by @rfc9334.
Each pattern, or an alternative, may be more suitable for particular types of activity.
For example, the passport model would be more suitable for a TEE providing a service to many unique users as long-lived attestation reports can be sent to many clients, without the need to an equal number of requests to the attestation service.

:::{important} Reliance on attestation services
Processing attestation evidence requires information which may be difficult to obtain.
In particular, verifying the authenticity of hardware requires a large set of keys matching those written to hardware at time of manufacturing.
This means that you must have trust in the hardware manufacturers.
Furthermore, it creates the possibility that this information may remain private, to protect the business of offering attention services, like Intel Trust Authority or cloud providers.
To organisations where TEEs are critical, there is a risk of

- Reliance on attestation providers, with no ability to host a local attestation service
- Hardware manufacturers controlling attestation and refusing to support competitors' hardware
- Withdrawal of service from attestation providers
or reliance on third parties for organisations where TEEs in critical.
:::

## Confidential Computing support

### Summary of the Current State of Confidential Computing

Confidential computing technology is still in a state of rapid development and prone to flux.
For example, AMD SEV technology has had a [number of extensions](https://www.amd.com/en/developer/sev.html) including SNP,
which includes important security improvements which could be considered essential for a TEE.
However, it has reached a state of maturity where the latest few generations of CPUs have support for robust and competent TREs.
This is mostly reserved for data-centre hardware, and not yet available on consumer devices.

Support utilising GPUs in TEEs is emerging, but less well developed that for CPUs.
Nvidia is currently the only vendor to offer GPU support in a TEE, and that is restricted to the lasted generation of Hopper devices.
This is discussed in [](#sec-ccs-vendor-nvidia-gpu)

(sec-ccs-cc-archetypes)=
#### TEE Archectypes

Two key archetypes for TEEs have emerged.

(sec-ccs-cc-archetypes-vm)=
##### Secure VMs

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
:::{mermaid}
block
  columns 3
  block:group_tee:2
    columns 2
    software data
    guest["guest OS"]
  end
  tee["TEE and TCB"]
  block:group_host:2
    columns 1
    hypervisor
    other["host software, drivers"]
    hostos["host OS"]
  end
  host["untrusted host components"]
  block:group_rot:2
    columns 1
    firmware["firmware and microcode"]
    cpu["CPU"]
    vendor["CPU vendor"]
  end
  rot["hardware root of trust"]

classDef label fill:#FFFFFF,stroke-width:0px;
class tee label
class host label
class rot label
classDef group fill:#FFFFFF,stroke-width:4px,stroke:#999999;
class group_tee group
class group_host group
class group_rot group
classDef trusted fill:#4DAF4A,stroke-width:0px;
class software trusted
class data trusted
class guest trusted
class cpu trusted
class firmware trusted
class vendor trusted
classDef untrusted fill:#377EB8,stroke-width:0px;
class hostos untrusted
class hypervisor untrusted
class other untrusted
:::
A block diagram showing the trusted and untrusted components of an example secure VM TEE.
::::

A secure VM TEE may provision a lower-trust portion of memory, which allows the transfer of data between the host and guest.

An advantage of the VM-based TEEs is their flexibility.
Little or no code changes will be required for software to run in such a TEE.
Building a secure application does not involve using CC APIs (possibly with the exception of moving data between host and guest) a confidentiality is guaranteed by the hardware and VM.

(sec-ccs-cc-archetypes-enclave)=
##### Enclaves

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
:::{mermaid}
block
  columns 3
  block:group_tee:2
    columns 2
    app_trusted["trusted app component\n and enclave"]
    app_untrusted["untrusted app component"]
  end
  tee["application"]
  block:group_host:2
    columns 1
    other["host software, drivers"]
    hostos["host OS"]
  end
  host["untrusted host components"]
  block:group_rot:2
    columns 1
    firmware["firmware and microcode"]
    cpu["CPU"]
    vendor["CPU vendor"]
  end
  rot["hardware root of trust"]

classDef label fill:#FFFFFF,stroke-width:0px;
class tee label
class host label
class rot label
classDef group fill:#FFFFFF,stroke-width:4px,stroke:#999999;
class group_tee group
class group_host group
class group_rot group
classDef trusted fill:#4DAF4A,stroke-width:0px;
class app_trusted trusted
class cpu trusted
class firmware trusted
class vendor trusted
classDef untrusted fill:#377EB8,stroke-width:0px;
class app_untrusted untrusted
class hostos untrusted
class other untrusted
:::
A block diagram showing the trusted and untrusted components of an example enclave TEE.
::::

Enclave TEEs have a smaller TCB compared to [VM-based TEEs](#sec-ccs-cc-archetypes-vm) as there is no guest OS, and only a small part of any application needs to be trusted.
This would make an in-depth security audit more feasible.
Developing for an enclave TEE will require calling CC APIs.
Translating an existing application would require splitting trusted routines and adding steps to create, attest and use enclaves to the untrusted part.

#### Confidential Containers

[Confidential Containers](https://confidentialcontainers.org/) is a project of the CNCF building open source tools to support running container applications in TEEs.
They aim to be hardware agnostic, supporting multiple TEE implementations and cloud service providers.
Software built by the community can support the whole lifecycle of a confidential container, provisioning, attestation and secret management.

There is [existing support](https://confidentialcontainers.org/docs/overview/#what-hardware-does-confidential-containers-support) for a number of different [TEE implementations](#sec-ccs-vendor) and archetypes.
Most vendors are supported through [Kata](https://katacontainers.io/), a container runtime which dispatches container processes to virtual machines, rather than convention sandboxes using Linux namespaces and cgroups.
Combining Kata with secure VMs, Confidential Containers allows a container engine (for example K8s) to run containers in confidential VMs.

(sec-ccs-vendor)=
### Vendor Support

<!-- Table (or similar summary) of the vendors and processors considered -->

:::{table} Vendor TEE implementations
:label: tab-vendor
| Vendor   | Hardware   | Archetype                                   | Implementation                              | Container Support[^cc]   |
| -------- | ---------- | ------------------------------------------- | ------------------------------------------- | ------------------------ |
| AMD      | CPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | SEV-SNP                                     | ✅ (with Kata)           |
| ARM      | CPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | CCA                                         | 🟠 (attestation only)    |
| Intel    | CPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | TDX                                         | ✅ (with Kata)           |
| Intel    | CPU        | [Enclave](#sec-ccs-cc-archetypes-enclave)   | SGX                                         | ✅                       |
| Nvidia   | GPU        | [Secure VM](#sec-ccs-cc-archetypes-vm)      | compatibile with SEV-SNP, CCA and TDX       |                          |
:::

[^cc]: Supported by the [Confidential Containers project](https://confidentialcontainers.org/docs/overview/)

#### AMD

(sec-ccs-vendor-amd-cpu)=
##### CPU

###### Summary

SEV-SNP is AMD's TEE implementation.
It is a [secure VM](#sec-ccs-cc-archetypes-vm) TEE.
SEV was the original secure virtualisation implementation.
There have been a number of additions to these original features and support [varies across CPU generations](https://www.amd.com/en/developer/sev.html).
In particular, SNP adds important memory protections for hypervisor-based attacks.
SEV is a TEE, but care should be taken as without SEV-SNP, there are known vulnerabilities which could be exploited by those with access to the hypervisor or host system.

Communication between host and secure guests is supported in SEV.
Each guest is able to specify which pages of memory should be private (encrypted with the guest's unique key) and shared (encrypted with a hypervisor key).

###### Hardware Support

TEEs are a feature of the datacentre EPYC line of CPUs.
SEV is supported from EPYC 7001 (Naples) and SEV-SNP from 7003 (Milan).
The [AMD website](https://www.amd.com/en/developer/sev.html) has a table of features introduced across subsequent CPU generations.

###### Resources

- [AMD Infinity Guard](https://www.amd.com/en/products/processors/server/epyc/infinity-guard.html) security technologies
- [AMD SEV](https://www.amd.com/en/products/processors/server/epyc/infinity-guard.html) developer information
- [SEV-SNP whitepaper](https://docs.amd.com/v/u/en-US/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more)
- [SEV-SNP attestation process](https://www.amd.com/content/dam/amd/en/documents/developer/lss-snp-attestation.pdf) presentation at Linux Security Summit 2022

##### GPU

There is currently no support for confidential computing with AMD GPUs.
There are signs that AMD considers secure computing on GPUs as an important, however there is no clear roadmap for its introduction.

> Security starts with the foundations:
>
> - Establishing a trusted execution environment by booting the device securely from power-on.
> - Ensuring supply chain security to verify the authenticity of devices, and
> - Adhering to industry standards and contributing to their development.
>
> On this foundation, we build advanced capabilities such as workload isolation through virtualization and confidential computing to help protect workloads during execution.
>
> Another key focus is assurance.
> At AMD, we implement a secure development lifecycle from design to release.
> This includes threat modeling to anticipate potential attacks and building countermeasures during the design phase.
>
> --- [Nathan Nadarajah](https://www.amd.com/en/blogs/2025/helping-secure-gpus-that-advance-ai.html)

#### ARM

(sec-ccs-arm-cpu)=
##### CPU

###### Summary

[CCA](https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture) is ARMs TEE implementation.
It is a secure VM approach, which depends on the strongly related RME technology, for managing isolated VMs (termed realms) on ARM platforms.
CCA is less mature than [SEV-SNP](#sec-ccs-vendor-amd-cpu) and [TDX](#sec-ccs-vendor-intel-cpu).
It is available on ARM datacentre CPUs but there is also an emphasis on making TEEs available on ARM-based mobile, automotive and embedded devices.

###### Hardware Support

CCA was introduced in with the RME instructions [ARMv9-A](https://www.arm.com/architecture/cpu/a-profile/armv9) in 2021.
The instructions may optionally be implemented on ARMv9-A CPUs
A number of Cotrex mobile and Neoverse datacentre CPUs have been developed with this or later, architectures.
The Neoverse V3 datacentre/cloud processors support CCA.
These have been manufactured into SoCs, AWS Graviton5, Azure Cobalt 200 and Nvidia Thor.

###### Resources

- [ARM CCA](https://www.arm.com/architecture/security-features/arm-confidential-compute-architecture) feature summary
- [CCA on Fujitsu Monaka](https://developer.arm.com/community/arm-community-blogs/b/servers-and-cloud-computing-blog/posts/how-fujitsu-implemented-confidential-computing-on-fujitsu-monaka-with-arm-cca) upcoming CPU blog post
- [RME developer guide](https://developer.arm.com/documentation/den0126/latest)

#### Intel

(sec-ccs-vendor-intel-cpu)=
##### CPU

###### Summary
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
The confidential VMs are refered to as trust domains.
Communication between the host and TEE is possible through a shared portion of memory, the Shared Extended Page Table.

###### Hardware Support

SGX was previously available on both consumer and datacentre processors, with support going back to the Skylake family in 2015.
Support has been removed in recent consumer devices, as Intel focuses more on developing TDX.
Support has remained for the latest generations of Xeron processors, with Intel indicating that they are committed to [continued support](https://community.intel.com/t5/Blogs/Products-and-Solutions/Security/Rising-to-the-Challenge-Data-Security-with-Intel-Confidential/post/1353141) on Xeon.

TDX is supported on the 5th and 6th Generation Xeon (Emerald Rapids and Granite Rapids respectively) CPUs.
Some cloud providers have special SKUs of 4th Generation Xeon (Sapphire Rapids) which are TDX-compatible, however these are not generally available.

###### Resources

- [SGX](https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html)
- [SGX Developers Guide](https://download.01.org/intel-sgx/latest/linux-latest/docs/Intel_SGX_Developer_Guide.pdf)
- [Intel SGX Use and Development Flow](https://www.intel.com/content/www/us/en/content-details/671417/intel-software-guard-extensions-and-the-development-flow-infographic.html)
- [TDX](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html)
- [Which Intel TEE is right for you](https://www.intel.com/content/www/us/en/content-details/818845/which-intel-trusted-execution-environment-is-right-for-you.html) graphic
- [Intel Trust Authority](https://www.intel.com/content/www/us/en/security/trust-authority.html) attestation service

##### GPU

Intel GPUs currently have no support for TEEs.

#### Nvidia

(sec-ccs-vendor-nvidia-gpu)=
##### GPU

###### Summary

For its recent GPU architectures, Nvidia has developed technology to allow its accelerators to be attested and used in secure guests.
Integration of Nvidia GPUs is supported for SEV-SNP, CMA and TDX.

###### Hardware Support

Confidential computing is supported on the Hopper and Blackwell architectures of datacentre GPUs.
It will also be supported on the future Rubin architecture.
When provisioned on a TEE capable host from a compatible vendor, a GPU from these families may be used by a secure VM.

:::{important}
Nvidia Grace Hopper superchips do not support confidential computing.
Despite packaging a Hopper GPU with an ARMv9 CPU, the CPU model (Neoverse V2) does not include RME instructions.
:::

The announced [Vera Rubin superchip](https://nvidianews.nvidia.com/news/rubin-platform-ai-supercomputer) will support confidential computing.
It is comprised of Rubin GPUs and Vera CPUs (with custom ARMv9-A cores) on a tightly integrated datacentre rack.

###### Resources

- [Nvidia confidential computing](https://www.nvidia.com/en-us/data-center/solutions/confidential-computing/)
- [H100 confidential computing](https://developer.nvidia.com/blog/confidential-computing-on-h100-gpus-for-secure-and-trustworthy-ai/) blog post
- [Vera Rubin](https://developer.nvidia.com/blog/inside-the-nvidia-rubin-platform-six-new-chips-one-ai-supercomputer/) blog post

## Availability

:::{table} Support for TEEs across various cloud and HPC platforms available in the UK
:label: tab-system
| Category   | System                                                               | TEE Support   | Details                                                                   |
| ---------- | -------------                                                        | ------------- | ------------------------------------------------------------------------- |
| AIRR       | Dawn                                                                 | ❌            | Old hardware generation (Intel CPUs)                                      |
| AIRR       | Isambard AI                                                          | ❌            | The GH200 superchips' Grace CPU does not support RME                      |
| Cloud      | AWS                                                                  | ✅            | [](#sec-availability-cloud-aws)                                           |
| Cloud      | Azure                                                                | ✅            | [](#sec-availability-cloud-azure)                                         |
| Cloud      | GCP                                                                  | ✅            | [](#sec-availability-cloud-gcp)                                           |
| STFC       | Mary Coombs                                                          | 🟠            | Hardware details not confirmed but will include H100s                     |
| Tier 1     | [ARCHER 2](https://www.archer2.ac.uk/about/hardware.html)            | 🟠            | CPUs with SEV (but not SNP) support                                       |
| Tier 2     | [Baskerville](https://docs.baskerville.ac.uk/system/)                | 🟠            | Very small number of nodes with H100s and AMD CPUs with SEV-SNP support   |
| Tier 2     | [CSD3](https://www.csd3.cam.ac.uk/high-performance-computing)        | ❌            | Old hardware generation (Intel CPUs and Nvidia GPUs)                      |
| Tier 2     | [Cirrus](https://www.cirrus.ac.uk/about/hardware-software/)          | ✅            | AMD CPUs with SEV-SNP support                                             |
| Tier 2     | [Kelvin 2](https://www.rc.ucl.ac.uk/docs/Clusters/Young/#node-types) | ✅            | Nodes supporting SEV, small number of nodes supporting SEV-SNP            |
| Tier 2     | [Sulis](https://sulis-hpc.github.io/techspecs/)                      | ✅            | Variety of nodes, including some with SEV and SEV-SNP support             |
| Tier 2     | [Young](https://www.rc.ucl.ac.uk/docs/Clusters/Young/#node-types)    | ❌            | SEV compatible processors but all nodes with incompatible GPUs            |
:::

### Cloud

(sec-availability-cloud-aws)=
#### AWS

AWS offers a number of infrastructure level security features as part of its [Nitro](https://aws.amazon.com/ec2/nitro/) hypervisor.
Among these are [confidential computing](https://aws.amazon.com/confidential-computing/) features,
including always-on memory encryption.
This feature protects users from people with hypervisor or hardware access.
It could be considered a TEE if applications were segregated by running on different instance, but this is not scalable solution.
To segregate data and software on the same host [Nitro enclaves](https://docs.aws.amazon.com/enclaves/latest/user/nitro-enclave.html), an AWS in-house enclave TEE implementation, can be used.
This extra isolation protects data from the customers own users and software, in addition to the default protection against AWS themselves.
Both of these Nitro features are available on AMD, ARM and Intel-based instances.

With compatible AMD instances, users can also opt to [enable SEV-SNP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/sev-snp.html).
This gives customers more flexibility to take over management of instance-specific encryption keys and attestation.
There is an extra charge on top of instance rate for enabling SEV-SNP.
Currently no GPU instances [support enabling SEV-SNP](https://docs.aws.amazon.com/ec2/latest/instancetypes/ac.html)

(sec-availability-cloud-azure)=
#### Azure

Azure offers a number of VM sizes [supporting confidential computing](https://learn.microsoft.com/en-us/azure/confidential-computing/overview-azure-products), and an attestation service.
SGX, TDX and SEV-SNP may be enabled on [compatible sizes](https://learn.microsoft.com/en-us/azure/confidential-computing/virtual-machine-options#sizes).
The NCCadsH100v5-series size support confidential computing with a GPU, combining an AMD EPYC Genoa CPU with an Nvidia H100 GPU.

SEV-SNP enabled VMs can be included in [AKS node pools](https://learn.microsoft.com/en-us/azure/confidential-computing/confidential-node-pool-aks)
Confidential VMs can also be used to back [some other services](https://learn.microsoft.com/en-us/azure/confidential-computing/overview-azure-products) like remote desktop and PostgreSQL.

(sec-availability-cloud-gcp)=
#### GCP

GCP has a variety of [confidential computing](https://cloud.google.com/security/products/confidential-computing#key-features) services.
[Confidential VMs](https://docs.cloud.google.com/confidential-computing/confidential-vm/docs/confidential-vm-overview) may be deployed with with Intel or AMD processors using [SEV](#sec-ccs-vendor-amd-cpu), [SEV-SNP](#sec-ccs-vendor-amd-cpu) or [TDX](#sec-ccs-vendor-intel-cpu) on [compatible sizes](https://docs.cloud.google.com/confidential-computing/confidential-vm/docs/supported-configurations#machine-type-cpu-zone).
One size compatible with confidential computing includes H100 GPUs.
Confidential VMs can be used as nodes in GKS Kubernetes.

<!-- ### Tier 1 HPC -->

<!-- #### ARCHER -->

<!-- - Each ARCHER node has two Intel E5-2697 v2 (Ivy Bridge) -->
<!-- - These do not support TDX -->

<!-- #### ARCHER2 -->

<!-- - https://www.archer2.ac.uk/about/hardware.html -->
<!-- - Each ARCHER2 node has 2 AMD EPYC 7742 -->
<!-- - The 7xx2 generation supports SEV but not SNP -->
<!-- - Should be sufficient for testing TEE containers, but lack of SNP does leave this vulnerable to certain kinds of attack -->

<!-- ### Tier 2 HPC -->

<!-- #### Cirrus -->

<!-- - https://www.cirrus.ac.uk/about/hardware-software/ -->
<!-- - Each node has 2 AMD EPYC 9825 -->
<!-- - Supports SEV-SNP with latest additional features -->
<!-- - Phase two adds Nvidia V100 GPUs not compatible with TEE -->

<!-- #### CSD3 -->

<!-- - https://www.csd3.cam.ac.uk/high-performance-computing -->
<!-- - CPU partitions are all too old to support TDX (Sapphire rapids or older) -->
<!-- - Wilkes 3 GPU cluster has Nvidia A100s which don't support confidential computing -->

<!-- #### Baskerville -->

<!-- https://docs.baskerville.ac.uk/system/ -->
<!-- H100s support confidential computing. -->
<!-- Baskerville has 2 H100 nodes (each with 4 GPUs and 2 AMD EPYC 9554 CPUs (4th generation)). -->
<!-- This is very similar to the confidential computing SKU offered on Azure. -->
<!-- AMD SEV-SNP and Nvidia GPU should work together. -->

<!-- Would be an interesting test case to work with Baskerville to see if SEV-SNP is enabled, or if we can work to enable it. -->

<!-- #### Sulis -->

<!-- - https://sulis-hpc.github.io/techspecs/ -->
<!-- - Variety of node types, suites Sulis' focus on HTC/anisotropic workflows -->
<!-- - Range of AMD EPYC processors 7xx2 (Rome) and 7xx3 (Milan) -->
<!-- - 7xx2 supports SEV, 7xx3 support SEV-SNP -->

<!-- #### MMM Hub Young -->

<!-- - https://www.rc.ucl.ac.uk/docs/Clusters/Young/#node-types -->
<!-- - GPU nodes have compatible AMD processors (7543), however the A100 GPUs do not support confidential computing -->
<!-- - Other nodes have sapphire rapids (6th gen) Xeon CPUs which do not support TDX -->

<!-- #### NIHPC Kelvin 2 -->

<!-- - https://ni-hpc.github.io/nihpc-documentation/Kelvin2%20Hardware/ -->
<!-- - CPU nodes with 2 AMD EPYC 7702 (supports SEV) -->
<!-- - 2 CPU nodes with 2 AMD EPYC 7773X (supports SEV-SNP) -->

## Conclusion

TEEs are not new technology, with implementations going back around a decade.
Despite that, it is still an active area of development and research.
Only the latest few generations of Intel and AMD processors support confidential VMs.
Furthermore in the case of Intel, a pivot from enclaves to secure guests fragments TEE support in their CPUs.
As a result, hardware support for TEEs is not common in UK research computing infrastructure.

As it stands, there is a little support for confidential computing in UK national-scale research computing.
The first generation of AIRR supercomputers, Dawn and Isambard-AI, both lack hardware supporting TEEs.
As the adoption of TEEs increases (and they become better integrated into tools for managing workloads, such as Kubernetes),
this may leave a gap in the ability to conduct research using sensitive data, particularly for AI tasks.

A number of Tier 2 systems have hardware compatible with secure virtualisation.
Mostly this is AMD SEV/SEV-SNP, but there is also hardware supporting TDX.
It is not clear if these systems have been configured to enable confidential VMs.
It seems unlikely as most of these systems are designed to be used through a scheduler (like SLURM) and not for users to deploy VMs.
Virtualisation may be disable altogether.
However, these systems could be used as TEE testbeds, perhaps especially as they approach end of service.

Currently, cloud providers fill that gap, with the largest services offering a choice of TEE implementation and supporting GPU use.
These resources may not be available to all research, for example when data governance imposes restrictions on the geography of data storage.
It also presents a presents challenges for researchers in managing costs and avoiding dependence on large-scale, private compute providers.
Better support from national resources could help enable research, promote the safe use of sensitive data in research and make research more financially efficient.

<!-- ## Glossary -->

<!-- :::{glossary} -->
<!-- Attestation -->
<!-- : … -->

<!-- CPU -->
<!-- : … -->

<!-- GPU -->
<!-- : … -->

<!-- Host -->
<!-- : … -->

<!-- Hypervisor -->
<!-- : … -->

<!-- Public-key Encryption -->
<!-- : … -->

<!-- Remote Attestation -->
<!-- : Attestation by a system other than the one hosting the TEE. This enables third parties to verify a TEE is valid. -->

<!-- Trust Domain -->
<!-- : … -->

<!-- Trusted Execution Environment -->
<!-- : … -->

<!-- Virtual Machine -->
<!-- : … -->
<!-- ::: -->
