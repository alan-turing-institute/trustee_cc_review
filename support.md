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
:::{image} https://mermaid.ink/img/pako:eNp9k8ty2yAUhl-FIVvFIwdf6SqxnW0703TTqtMhErpMJNAAcpx6_O49gIRjNY424nzn9gOHI05lxjHFz7VMXxKBUCrrrhEaEWs4Sgslu_aP4ZzeWXiO6U0tc_PKFEcZM8yjouPa_Eqw-6Ov3xP82zq4yOwPSoHvabdDTGToafPQu9-3K6U2435Tb5ZvLVf7SkvlbWlKrqCgTQliIpSpas-V7mtDGrilHuLGmiwEXyeM6rThmQPQuWml4MLoDyQqeU1hXqnGioCCw9JttalSJe2JB1Fp20HQ5tuPQPYgSCoPe-NSKbS1m2Aqc3WVlAbJHDndLjIRac203vIc1eyZ1yCnrunNo_sibZR84bevVWZKGreHL324vRUfPwB3AhcEWl8A28KdxSctZu2hB_Rm7b7QMQyWX11i1_0DbjW8x1bDcGdexWx7_zi7v77RMK992sDt-I6ZH-ARhEsbo3DNI-4vcESt5POgedFkudw9rK6L9tN7Tgs8PIb_fe5hnDGOcKGqDFOweYQbrhpmTXy0Y5VgiG5gMiksM56zroZxSsQJ0lomfkrZDJlw-kWJac5qDVbXwsHxbcUKxZpAFWycq42E7piS2NXA9IgP1iKTmCymC0JIHC9X0wi_YTpfTaar5TxeLu7WZD2bk1OE_7qm8QT46R8dq6MJ?type=png
:::
% :::{mermaid}
% block
%   columns 3
%   block:group_tee:2
%     columns 2
%     software data
%     guest["guest OS"]
%   end
%   tee["TEE and TCB"]
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
%     vendor["CPU vendor"]
%   end
%   rot["hardware root of trust"]
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
% class vendor trusted
% classDef untrusted fill:#377EB8,stroke-width:0px;
% class hostos untrusted
% class hypervisor untrusted
% class other untrusted
% :::
The trusted and untrusted components of an example secure VM TEE.
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
:::{image} https://mermaid.ink/img/pako:eNp9k8tu2zAQRX9FYLaqYb0tdpXGzbYFim5aFQUjUbYQiSOQlOPW8L93SL0sNYk25lze4RxyxheSQ8EJJU815M-ZcJwc6q4RyglMYFV6kNC1vzXn1Dfi7BlC1uKu7JTmxc-MDCujorFpQXChM3QyUThc5DU78Yz8mlM7MSdP61V67-eiMD9IglY01FXOdAVi2L6lPYLSa1yvD0EfucQDjMVRUOoXJrnrFLI6cakmNLMNavR9-bZkMOKC17omYPUKkoS3iMpKNgYCDxyX9rWaKpdgGjRB5W2Hpoev3yflhEAge3EIlqRY1lyCycKeKwG0A6Vjua0zMz1Ras9Lp2ZPvEacuqZ3j_ZzlZbwzD-8VIU-0m17_jjYTRd6_yjYF1goWHohmBL2Ld4pEbbnQaB3qf2mitMc9qulbKu_ohuGW9kwjD3rKcL9_WN4__ZFb8Z7zBy3sB1raWrgSu9bs1INzDxCPU6QJJ8_7d7HmXOm1W0XQP2v26mfZeKSg6wKQjHmLmm4bJgJycXMDP6Lj7zBsaO4LHjJuhpnJRNXTGuZ-AHQjJn4tIcjoSWrFUZdWzDN9xU7SNZMqsS7c_kAWJ1QP47tIYReyJlQL_E3URgE0db30sSPI9z9Q2gYeJt0t0t93_NSL9zGV5f8tWW3m2QbpUkShOEuiYIgjq__AJGorRk?type=png
:::
% :::{mermaid}
% block
%   columns 3
%   block:group_tee:2
%     columns 2
%     app_trusted["trusted app component\n and enclave"]
%     app_untrusted["untrusted app component"]
%   end
%   tee["application"]
%   block:group_host:2
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
% class tee label
% class host label
% class rot label
% classDef group fill:#FFFFFF,stroke-width:4px,stroke:#999999;
% class group_tee group
% class group_host group
% class group_rot group
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
The trusted and untrusted components of an example enclave TEE.
::::

Enclave TEEs have a smaller TCB compared to [VM-based TEEs](#sec-ccs-cc-archetypes-vm) as there is no guest OS, and only a small part of any application needs to be trusted.
This would make an in-depth security audit more feasible.
Developing for an enclave TEE will require calling CC APIs.
Translating an existing application would require splitting trusted routines and adding steps to create, attest and use enclaves to the untrusted part.

### Confidential Containers

[Confidential Containers](https://confidentialcontainers.org/) is a project of the CNCF building open source tools to support running container applications in TEEs.
They aim to be hardware agnostic, supporting multiple TEE implementations and cloud service providers.
Software built by the community can support the whole lifecycle of a confidential container, provisioning, attestation and secret management.

There is [existing support](https://confidentialcontainers.org/docs/overview/#what-hardware-does-confidential-containers-support) for a number of different [TEE implementations](#sec-ccs-vendor) and archetypes.
Most vendors are supported through [Kata](https://katacontainers.io/), a container runtime which dispatches container processes to virtual machines, rather than convention sandboxes using Linux namespaces and cgroups.
Combining Kata with secure VMs, Confidential Containers allows a container engine (for example K8s) to run containers in confidential VMs.

(sec-ccs-vendor)=
## Vendor Support

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

##### Hardware Support

TEEs are a feature of the datacentre EPYC line of CPUs.
SEV is supported from EPYC 7001 (Naples) and SEV-SNP from 7003 (Milan).
The [AMD website](https://www.amd.com/en/developer/sev.html) has a table of features introduced across subsequent CPU generations.

##### Resources

- [AMD Infinity Guard](https://www.amd.com/en/products/processors/server/epyc/infinity-guard.html) security technologies
- [AMD SEV](https://www.amd.com/en/products/processors/server/epyc/infinity-guard.html) developer information
- [SEV-SNP whitepaper](https://docs.amd.com/v/u/en-US/SEV-SNP-strengthening-vm-isolation-with-integrity-protection-and-more)
- [SEV-SNP attestation process](https://www.amd.com/content/dam/amd/en/documents/developer/lss-snp-attestation.pdf) presentation at Linux Security Summit 2022

#### GPU

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

##### Resources

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
The confidential VMs are refered to as trust domains.
Communication between the host and TEE is possible through a shared portion of memory, the Shared Extended Page Table.

##### Hardware Support

SGX was previously available on both consumer and datacentre processors, with support going back to the Skylake family in 2015.
Support has been removed in recent consumer devices, as Intel focuses more on developing TDX.
Support has remained for the latest generations of Xeron processors, with Intel indicating that they are committed to [continued support](https://community.intel.com/t5/Blogs/Products-and-Solutions/Security/Rising-to-the-Challenge-Data-Security-with-Intel-Confidential/post/1353141) on Xeon.

TDX is supported on the 5th and 6th Generation Xeon (Emerald Rapids and Granite Rapids respectively) CPUs.
Some cloud providers have special SKUs of 4th Generation Xeon (Sapphire Rapids) which are TDX-compatible, however these are not generally available.

##### Resources

- [SGX](https://www.intel.com/content/www/us/en/developer/tools/software-guard-extensions/overview.html)
- [SGX Developers Guide](https://download.01.org/intel-sgx/latest/linux-latest/docs/Intel_SGX_Developer_Guide.pdf)
- [Intel SGX Use and Development Flow](https://www.intel.com/content/www/us/en/content-details/671417/intel-software-guard-extensions-and-the-development-flow-infographic.html)
- [TDX](https://www.intel.com/content/www/us/en/developer/tools/trust-domain-extensions/overview.html)
- [Which Intel TEE is right for you](https://www.intel.com/content/www/us/en/content-details/818845/which-intel-trusted-execution-environment-is-right-for-you.html) graphic
- [Intel Trust Authority](https://www.intel.com/content/www/us/en/security/trust-authority.html) attestation service

#### GPU

Intel GPUs currently have no support for TEEs.

### Nvidia

(sec-ccs-vendor-nvidia-gpu)=
#### GPU

##### Summary

For its recent GPU architectures, Nvidia has developed technology to allow its accelerators to be attested and used in secure guests.
Integration of Nvidia GPUs is supported for SEV-SNP, CMA and TDX.

##### Hardware Support

Confidential computing is supported on the Hopper and Blackwell architectures of datacentre GPUs.
It will also be supported on the future Rubin architecture.
When provisioned on a TEE capable host from a compatible vendor, a GPU from these families may be used by a secure VM.

:::{important}
Nvidia Grace Hopper superchips do not support confidential computing.
Despite packaging a Hopper GPU with an ARMv9 CPU, the CPU model (Neoverse V2) does not include RME instructions.
:::

The announced [Vera Rubin superchip](https://nvidianews.nvidia.com/news/rubin-platform-ai-supercomputer) will support confidential computing.
It is comprised of Rubin GPUs and Vera CPUs (with custom ARMv9-A cores) on a tightly integrated datacentre rack.

##### Resources

- [Nvidia confidential computing](https://www.nvidia.com/en-us/data-center/solutions/confidential-computing/)
- [H100 confidential computing](https://developer.nvidia.com/blog/confidential-computing-on-h100-gpus-for-secure-and-trustworthy-ai/) blog post
- [Vera Rubin](https://developer.nvidia.com/blog/inside-the-nvidia-rubin-platform-six-new-chips-one-ai-supercomputer/) blog post


