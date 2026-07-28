---
title: Confidential Computing Availability
abstract: |
  An evaluation of the availability of confidential computing in UK research infrastructure
authors:
  - name: Jim Madge
    orcid: 0000-0001-6044-164X
license: CC-BY-4.0
keywords:
    - confidential computing
    - trusted execution environments
    - high performance computing
    - cloud computing
    - trusted research
---

## Summary

:::{table} Support for TEEs across various cloud and HPC platforms available in the UK
:label: tab-system
| Category   | System                                                               | TEE Support   | Details                                                                   |
| ---------- | -------------                                                        | ------------- | ------------------------------------------------------------------------- |
| AIRR       | Dawn                                                                 | ❌            | Pre-TDX Intel CPU generation                                              |
| AIRR       | Isambard AI                                                          | ❌            | The GH200 superchips' Grace CPU does not support RME                      |
| Cloud      | AWS                                                                  | ✅            | [](#sec-availability-cloud-aws)                                           |
| Cloud      | Azure                                                                | ✅            | [](#sec-availability-cloud-azure)                                         |
| Cloud      | GCP                                                                  | ✅            | [](#sec-availability-cloud-gcp)                                           |
| STFC       | Mary Coombs                                                          | 🟠            | Hardware details not confirmed but will include H100s                     |
| Tier 1     | [ARCHER 2](https://www.archer2.ac.uk/about/hardware.html)            | 🟠            | CPUs with SEV (but not SNP) support                                       |
| Tier 2     | [Baskerville](https://docs.baskerville.ac.uk/system/)                | 🟠            | Very small number of nodes with H100s and AMD CPUs with SEV-SNP support   |
| Tier 2     | [CSD3](https://www.csd3.cam.ac.uk/high-performance-computing)        | ❌            | Pre-TDX Intel CPU generation                                              |
| Tier 2     | [Cirrus](https://www.cirrus.ac.uk/about/hardware-software/)          | ✅            | AMD CPUs with SEV-SNP support                                             |
| Tier 2     | [Kelvin 2](https://www.rc.ucl.ac.uk/docs/Clusters/Young/#node-types) | ✅            | Nodes supporting SEV, small number of nodes supporting SEV-SNP            |
| Tier 2     | [Sulis](https://sulis-hpc.github.io/techspecs/)                      | ✅            | Variety of nodes, including some with SEV and SEV-SNP support             |
| Tier 2     | [Young](https://www.rc.ucl.ac.uk/docs/Clusters/Young/#node-types)    | ❌            | CPUs with SEV (but not SNP) support, incompatible GPUs                    |
:::

## Cloud

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
