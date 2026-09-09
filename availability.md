---
title: Confidential Computing Availability
abstract: |
  An evaluation of the availability of confidential computing in UK research infrastructure
authors:
  - name: Jim Madge
    orcid: 0000-0001-6044-164X
  - name: Duncan Leggat
    orcid: 0009-0007-2922-3610
license: CC-BY-4.0
keywords:
    - confidential computing
    - trusted execution environments
    - high performance computing
    - cloud computing
    - trusted research
---

## Availability Summary

- ✅ full support
- 🟠 partial support
- ❌ no or minimal support
- ❓ unknown

:::{table} Support for TEEs across various cloud and HPC platforms available in the UK
:label: tab-system
| Category | System                                                               | TEE Compatible Hardware | TEE Offering | Details                                                                 |
|----------|----------------------------------------------------------------------|-------------------------|--------------|-------------------------------------------------------------------------|
| AIRR     | Dawn                                                                 | ❌                      | ❌           | Pre-TDX Intel CPU generation                                            |
| AIRR     | Zenith                                                               | ❓                      | ❓           |                                                                         |
| AIRR     | Isambard AI                                                          | ❌                      | ❌           | The GH200 superchips' Grace CPU does not support RME                    |
| Cloud    | AWS                                                                  | ✅                      | ✅           | [](#sec-availability-cloud-aws)                                         |
| Cloud    | Azure                                                                | ✅                      | ✅           | [](#sec-availability-cloud-azure)                                       |
| Cloud    | GCP                                                                  | ✅                      | ✅           | [](#sec-availability-cloud-gcp)                                         |
| STFC     | Mary Coombs                                                          | 🟠                      | ❓           | Hardware details not confirmed but will include H100s                   |
| Tier 1   | [ARCHER 2](https://www.archer2.ac.uk/about/hardware.html)            | 🟠                      | ❌           | CPUs with SEV (but not SNP) support                                     |
| Tier 2   | [Baskerville](https://docs.baskerville.ac.uk/system/)                | 🟠                      | ❌           | Very small number of nodes with H100s and AMD CPUs with SEV-SNP support |
| Tier 2   | [CSD3](https://www.csd3.cam.ac.uk/high-performance-computing)        | ❌                      | ❌           | Pre-TDX Intel CPU generation                                            |
| Tier 2   | [Cirrus](https://www.cirrus.ac.uk/about/hardware-software/)          | ✅                      | ❌           | AMD CPUs with SEV-SNP support                                           |
| Tier 2   | [Kelvin 2](https://www.rc.ucl.ac.uk/docs/Clusters/Young/#node-types) | ✅                      | ❌           | Nodes supporting SEV, small number of nodes supporting SEV-SNP          |
| Tier 2   | [Sulis](https://sulis-hpc.github.io/techspecs/)                      | ✅                      | ❌           | Variety of nodes, including some with SEV and SEV-SNP support           |
| Tier 2   | [Young](https://www.rc.ucl.ac.uk/docs/Clusters/Young/#node-types)    | ❌                      | ❌           | CPUs with SEV (but not SNP) support, incompatible GPUs                  |
:::

## Public Cloud

(sec-availability-cloud-aws)=
### AWS

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
### Azure

Azure offers a number of VM sizes [supporting confidential computing](https://learn.microsoft.com/en-us/azure/confidential-computing/overview-azure-products), and an attestation service.
SGX, TDX and SEV-SNP may be enabled on [compatible sizes](https://learn.microsoft.com/en-us/azure/confidential-computing/virtual-machine-options#sizes).
The NCCadsH100v5-series size support confidential computing with a GPU, combining an AMD EPYC Genoa CPU with an Nvidia H100 GPU.

SEV-SNP enabled VMs can be included in [AKS node pools](https://learn.microsoft.com/en-us/azure/confidential-computing/confidential-node-pool-aks)
Confidential VMs can also be used to back [some other services](https://learn.microsoft.com/en-us/azure/confidential-computing/overview-azure-products) like remote desktop and PostgreSQL.

(sec-availability-cloud-gcp)=
### GCP

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

(sec-av-challenges)=
## Challenges in Adoption in HPC

The hardware of many modern HPC systems support confidential computing and TEEs.
The proportion of compatible clusters will increase as older generations of hardware are decommissioned and replaced.
However, even in cases where hardware would allow it, CC and secure virtualisation are not available to users.

Enabling CC presents a number of challenges for the administrators of HPC systems,
which prioritise stability[^stability], reliability[^reliability], uptime[^uptime], performance[^performance] and throughput[^throughput] to maximise their usage and output.

[^stability]: A consistent state which does not change, for example maintaining ABI compatibility.
  This is not the same as reliability.
[^reliability]: A state of being bug and problem free.
[^uptime]: Time that a system spend running and available for work.
[^performance]: How quickly a computer can perform calculations, measured in FLOPS.
[^throughput]: The overall rate of output of a computer across all tasks, in contrast to the peak calculation rate for a single task.
  Throughput therefore depends on how efficient resources can be utilised across all jobs submitted and depends on the effective scheduling of tasks as well as raw performance.

Activating the CPU and {term}`secure processor` features that enable CC is done through UEFI.
In a HPC system these changes must be made on every node that must support confidential computing, potentially hundreds or thousands of nodes.
More problematic that UEFI configuration is managing the OS and kernel versions across nodes.
This is necessary as TEE implementations will require compatible kernels, and hence OS.

Changing the kernel or OS version on a HPC system risks introducing bugs or breaking existing hardware.
HPC systems tends to lean towards LTS[^LTS] kernels for stability, which may lack support for the CC features of new hardware.
For example, at the time of writing, Rocky Linux 9 and RHEL 9 (popular choices for HPC) by contrast, operate on Linux kernel 5.14, which does not support CC.
The latest release, RHEL 10 (updated to kernel 6.12) offers [support for CVMs](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/10.0_release_notes/technology-preview-features#technology-preview-features-virtualization) as a [technology preview](https://access.redhat.com/support/offerings/techpreview).
Technology preview features are not fully supported and are intended for production use.
So, with the latest RHEL release operators would take on risk in using a feature that is not fully supported and without guarantee.

[^LTS]: Long Term Support kernels are maintained branches of the Linux kernel behind the latest version,
  which incorporate bug fixes from more recent releases.

Furthermore, HPC systems may rely on drivers for high performance hardware (parallel storage, fast networking, accelerators) which impose restrictions on kernel versions.
It is therefore possible that administrators must choose between CC support or hardware features when deciding on a kernel version.
In these cases the optimal functioning of hardware is likely to win out over a relatively niche feature.

Both UEFI and kernel/OS configuration steps would require downtime to implement on an existing system.
Kernel/OS changes in particular could result in extended downtime as it may additionally require rebuilding software, updating drivers, or rewriting scripts in reaction to CLI changes.

The data protection afforded by TEEs comes with a negative impact on performance.
Performance loss typically varies between [2% and 10%](https://www.servnetuk.com/learn/confidential-computing-explained#s-8) compared to conventional VMs.
The performance difference depends strongly on the workflow, with memory-intensive work suffering higher losses than CPU-bound tasks [@xinyuan-benchmarking; @coppolino-experimental].
The performance gap between confidential and conventional workloads will likely decrease as TEEs are further developred.
However, the performance loss and increased memory latency will likely prevent it being used by default.
For users the performance cost may be an acceptable to enabled trusted research on an HPC system.
However, for HPC operators a significant number of confidential jobs would reduce the throughput of the cluster compared to only non-confidential processes.

Additionally, many HPC systems do not support virtualisation at all.
For most jobs, creating a virtualmachine would be unnecessary and only result in longer stand-up and run times.
Even when it is, the scheduling of CVMs is also a challenge.
A simple way to handle this would be to create persistent CVMs, which are handed over to users, who then manage [attestation](#sec-cc-attestation) and work interactively in the TEE.
However, this will lead to the allocated resources spending much of their time idle and unavailable to other HPC users.
Alternatively, CVMs could be created dynamically by the scheduler in response to demand, but that presents new challenges in building infrastructure to manage the secure release of confidential data, the workloads and secrets to the CVM.
This is similar to the set of challenges the [Confidential Containers](https://confidentialcontainers.org/) project addresses for Kuberentes.

## Conclusion

As it stands, there is a little support for confidential computing in UK national-scale research computing.
The first generation of AIRR supercomputers, Dawn and Isambard-AI, both lack hardware supporting TEEs.
A number of Tier 2 systems have hardware compatible with secure virtualisation.
However, none offer CC to users.
Significant [challenges](#sec-av-challenges) remain in how to support CC in the HPC context.
As the adoption of TEEs increases, this may leave a gap in the ability to conduct research using sensitive data, particularly for AI tasks.

Currently, cloud providers fill that gap, with the largest services offering a choice of TEE implementations and hardware configurations (including GPUs).
These resources may not be available to all research, for example when data governance imposes restrictions on the geography of data storage.
It also presents a presents challenges for researchers in managing costs and avoiding dependence on large-scale, private compute providers.
Better support from national resources could help enable research, promote the safe use of sensitive data in research and make research more financially efficient.

If there is critical need for CC on the next generation of HPC machines, it must be influence the design of the system from conception.
Beginning at procurement, hardware that supports CC must be chosen.
CC support must also factor into kernel and OS choice, alongside constraints from other hardware and considerations of stability and software support.
Beyond that, there is a significant unsolved problem of how to schedule and manage TEEs in a manner suitable for HPC.
At present, with the lack of an open project similar to [Confidential Containers](https://confidentialcontainers.org/) for HPC,
early adopters may find they have to implement this supporting infrastructure themselves.
