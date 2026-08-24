---
title: Trusted Execution Environments for Trusted Research Environments
abstract: |
  An evaluation of the costs and benefits of integrating TEEs with TREs.
authors:
  - name: Jim Madge
    orcid: 0000-0001-6044-164X
license: CC-BY-4.0
keywords:
  - confidential computing
  - trusted execution environments
  - trusted research
  - trusted research environments
---

## Introduction

Trusted execution environments provide an unrivaled level of privacy when using a computer.
They fill an critical security gap, by keeping data secure while in use through encryption.
This is in addition to existing methods to encrypt data at rest and while in transit.

Trusted research environments play an important role in research, allowing researchers to work with sensitive data in a way that is effective, efficient and secure.
Integrating TEEs in to TREs could open new possibilities for trusted research, both in terms of the kinds of data we can use for research and for where that research can be conducted.
In particular, the strong isolation of a TEE from the rest of the computer could enable us to deploy TREs to third-party infrastructure (such as supercomputers or cloud services) or run them on untrusted devices (such as personal laptops).

However, using TEEs is not trivial and requires careful design of policy, compliance checks and investment in hardware.
Furthermore, though the isolation of a TEE from the rest of the computer is strong, all processes in the TEE must be trusted, so we must think critically about what a TEE does and does not protect you against.
Here we evaluate the situations where TEEs add value to TREs considering the costs against the benefits for TRE security.

:::{embed} #sec-cc-tees
:::

(sec-tre-security)=
:::{embed} #sec-cc-security
:::

:::{important} TRE security
In TREs, it is appropriate to use non-technical controls to build trust and ensure security.
There are many options available to manage risk and the use of a TRE does not mean that other strategies can be neglected or are no longer needed.
This is especially true in cases where technical solutions are impractical or complex.
A more holistic view of security when handling sensitive data is described by [The Five Safes](https://ukdataservice.ac.uk/help/secure-lab/what-is-the-five-safes-framework/)
:::

## Roles and entities

The language used to describe TRE systems varies.
To help clarify the [recommendations](#sec-recommendations) the following roles and entity definitions are used.

### Entities

::: {glossary}
TRE Project
: A piece of sensitive work with a single set of governance outlining its rules.
  This could be a research project addressing a particular question, or the curation of an important data set.

TRE Implementation
: The software/infrastructure that enables a {term}`TRE` enforcing (aspects of) security controls and governance rules,
  while providing an effective workspace for research.

TRE
: The environment used for the purpose of a {term}`TRE project`, with rules for usage defined by a governance document.
  This will often comprise of a single deployment of a {term}`TRE implementation`, governance, processes and support.

TRE Infrastructure
: The underlying platform on which a {term}`TRE` is deployed.
  For example, private or public cloud, an on-premise server, or an HPC cluster.
:::

### Roles

:::{glossary}
TRE Operator
: Manages {term}`TRE`.
  For example controlling user access, managing data ingress and egress, enforcing governance.

Infrastructure Operator
: Manages the underlying infrastructure on which {term}`TRE` is deployed.
  May be responsible for configuration which enforces TRE security, such as network controls or deploying CVMs.

TRE Developer
: Builds {term}`TRE Implementation`.

TRE User
: Authorised to use a {term}`TRE`.
:::

(sec-recommendations)=
## Recommendations

By considering the design of TREs and [the scope of TEEs protection](#sec-cc-security) we have arrived at the following situations where TEEs add substantial value to TREs and should be considered.

(sec-recommendations-isolation)=
### 1. When strong isolation from the host is required

Providing a way for compute to be run on an untrusted host, or host with untrusted users, is the central goal of confidential computing.
This aligns well with many existing TRE scenarios, particularly where TREs are deployed to systems not entirely dedicated to trusted research (for example, local HPC or private cloud), or where a TRE is deployed to a third-party's system.
In these cases, although there is likely a high level of trust in the host, protection from attacks targeting the host or vulnerabilities on the host infrastructure is valuable.
Only when the same party holds the {term}`TRE operator` and {term}`infrastructure operator` roles _and_ the {term}`TRE infrastructure` is dedicated
(that is, it isn't used for non-sensitive work) is isolation from the host not a strong benefit.

:::{important} Isolating jobs or users
TEEs are _not_ necessary for isolating individual jobs or {term}`TRE projects <TRE project>` from each other.
This may be important in a {term}`TRE implementation` which supports multiple {term}`TRE projects <TRE project>` using the same {term}`TRE infrastructure`.
In these cases, isolation can be introduced by using VMs.
Since no {term}`TRE user` has no access to the host or hypervisor, the attacks which TEEs protect against are not viable.
:::

#### 1.a. TRE operator and infrastructure operator are different parties

When the {term}`TRE operator` and {term}`infrastructure operator` roles are not held by the same party, the {term}`TRE infrastructure` is _not_ controlled by the {term}`TRE operator`.
The {term}`infrastructure operator` will have a high degree of privilege and access to the {term}`TRE infrastructure`, which potentially allows them to observe sensitive processes in breach of TRE governance.
The cryptographic isolation of a TRE from the {term}`TRE infrastructure` using a TEE ensures that the {term}`infrastructure operator` is not able to access data in use, whether maliciously or accidentically.

Precisely how important this is will depend on the trust in the infrastructure operator and the guarantees they provide.
For example, large cloud providers rely on their ability to isolate tenant from each other, and from the cloud provider's staff to operate.
Their processes are documented and audited as part of their certification and compliance with information security standards, for example [AWS](https://aws.amazon.com/compliance/services-in-scope/), [Azure](https://learn.microsoft.com/en-us/azure/compliance/offerings/) and [GCP](https://cloud.google.com/security/compliance/offerings#/).
In particular, the [ISO/IEC 27017](https://www.iso.org/standard/27017) standard (part of the [ISO/IEC 27000 family](https://www.iso.org/standard/iso-iec-27000-family)) focuses on the challenges of cloud services including isolating tenants environments.

However, even in cases where there is absolute trust in the {term}`infrastructure operator` TREs are still vulnerable to attack from a compromised {term}`TRE infrastructure`.
Other scenarios where TEEs provide valuable isolation to TREs more appropriate solution are,

- Satellite TREs using a shared responsibility model (such as [FRIDGE](xref:fridge) deployed on HPC)
- Federated research where one {term}`TRE operator` runs a workload or deploys an ephemeral TRE within another TRE

#### 1.b. TRE infrastructure is not exclusive

When the {term}`TRE infrastructure` is not used exclusively for trusted research, the risk of non-authorised administrators accessing sensitive data (whether accidentally or maliciously) is raised.
In this scenario, different rules and processes for the administration of sensitive and non-sensitive environments could lead to errors.
An example of this would be an HPC system which provides a service for both sensitive and non-sensitive workloads.
This model could be particularly effective on a private cloud, a {term}`TRE project` would be allocated a CVM while non-sensitive work is conducted with conventional VMs.

(sec-recommendations-lowtrust)=
### 2. When using low trust devices

Another possible scenario is low-trust, end-user devices or "bring your own compute".
As a TEE does not depend on trust in the host operating system or software, you can use a TEE to run trusted computation on an untrusted device.

:::{important} Enabling new models of trusted research
Unlike the situations in [class 1](#sec-recommendations-isolation), where the host is trusted but vulnerable, in this case the devices are _always_ untrusted.
The use of TEEs is not only an improvement in security but critical to conducting trusted research on untrusted devices.
TEEs are therefore an enabling technology for new models of trusted research.
:::

For example, individual laptops or institutional servers which are not designed for secure, multi-tenant use.[^commercial-use]
In the extreme, widespread support of TEEs could allow distributed trusted research across untrusted devices, which would currently be far too risky to consider.
Perhaps this could scale to large pools of workers similarly to how non-sensitive distributed research has been conducted by [Folding@Home](https://foldingathome.org/) or SETI@Home.

[^commercial-use]: This is perhaps closer to the applications of confidential computing outside of trusted research, where a key challenge is running a process on an untrusted machine while ensuring its integrity and confidentiality.

(sec-recommendations-sensitive)=
### 3. When highly sensitive data must be used

When dealing with highly-sensitive data, computer security is not the only option for protecting against unauthorised disclosure.
Instead, in line with the Five Safes framework [@five-safes], a more holistic approach to security should be taken and the data should be modified to reduce disclosure risk if possible (for example pseudononymisation, data minimisation or the use of synthetic or dummy data).

When highly-sensitive data must be used, however, it is appropriate to reduce risk in other dimensions.
In these cases, the use of TEEs may be beneficial.
It would offer most benefit when dealing with data which would encourage high-motivated attackers to launch sophisticated attacks targeted at the {term}`TRE infrastructure`.
For example, compromising the host OS, BIOS or a social engineering attack targeting the {term}`infrastructure operator`.

:::{attention} Viability of attack
LLM agents have been demonstrated finding and exploiting zero-day vulnerabilities in computer systems.
As models improve, particularly open-weight models which can be run locally without guard rails, so will their ability to launch successful attacks.
And so, while the attacks TEEs mitigate are complex, the barrier to launch will lower.
In there future the threshold of data sensitivity (or value) where TEEs become necessary may reduce.
:::

(sec-considerations)=
## Considerations

The [above recommendations](#sec-recommendations) outline the situations when TEEs add value to a TRE, in terms of the TREs design and usage.
However, the decision of whether to use TEEs must also include a consideration of the costs involved, so that an assessment of the net benefit can be made.

(sec-considerations-hardware)=
### Hardware support and availability

Although confidential computing is not new, it has not reached a maturity where all or most devices can be used to provision TEEs.
For CVM solutions, only the most recent, perhaps two or three generations, of devices will have support.
Furthermore, as the technology is still evolving, older CPU generations may lack the features, and enhanced security, or more modern processors.
TEE support is often only available in datacentre CPUs and not present on consumer devices.

Integration of devices into TEEs is not generally solved, although the latest generations of GPUs from Nvidia and AMD can be incorporated into a TEE.
For many, it may therefore not be possible to build on TEEs without a large investment in new hardware.

(sec-considerations-configuration)=
### TEE and host configuration

In addition to capital investment in hardware, an organisation implementing confidential computing on-premises will need to perform the necessary configuration and setup.
Vendor guides give an impression of the work involved, for example these setup guides from [AMD](https://docs.amd.com/v/u/en-US/58207-using-sev-with-amd-epyc-processors) and [Intel](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/01/introduction/).
This may include,

- BIOS (UEFI) configuration
- Installing a compatible Host OS
- Host OS configuration
- Guest OS configuration (for example, building a minimal guest image with necessary tools)
- Configuring other trusted hardware, such as GPUs or NICs

Configuration costs may be avoided by using a third-party, such as a cloud computing provider, to provision TEEs as a service.
This may however be a trade in upfront expense for larger operational expenses.

### TRE design and implementation cost

There is a tremendous diversity in {term}`TRE implementations <TRE implementation>`.
It is still common for organisations to build or commission their own {term}`TRE` and they are often designed for the needs of one organisation.
As such, despite the emergence of archetypes and common tools, there is no standard {term}`TRE implementation` or architecture.
This makes it difficult to make blanket statements about how TEEs should be used in {term}`TRE implementations <TRE implementation>`.
Incorporating TEEs into a {term}`TRE implementation` is not a trivial change.

TRE builders should take care to consider how and where to use confidential computing to protect sensitive workloads inside a {term}`TRE`
A key consideration is how the use of a TEE affects the size of the [TCB](#tip-tcb).
The focus for TRE developers should be to,

- Move sensitive processes into TEEs first
- Minimise the size of TEEs (and hence the [TCB](#tip-tcb))

Moving large, monolithic TRE components into a TEE may not produce much benefit.
As [explained above](#sec-tre-security), all code moved into a TEE remains part of the [TCB](#tip-tcb) and vulnerabilities in the [TCB](#tip-tcb) will compromise security, irrespective of the secure TEE boundary.
For example, moving large "workspace VMs" to CVMs, while providing isolation from the host, will not provide any mitigation for vulnerabilities in the workspace OS or software.

With a clear design the development effort or introducing TEEs can be assessed.
In some cases, it may be a simple task of shifting existing entities from VMs to CVMs, or pods from a conventional runner to [confidential containers](#sec-ccs-coco).
However, in other cases it may require significant code changes like the splitting of confidential and non-confidential code, building on CC APIs, or major architectural changes to the {term}`TRE implementation`.

(sec-considerations-infra)=
### Attestation infrastructure

Enabling secure virtualisation should give a high level of confidentiality from the host, whether or not the state of a TEE is verified through attestation.
However, by skipping the attestation, systems which interact with the TEE (and may rely on it being secure and confidential) cannot determine where the TEE is trustworthy.[^performative-security]
@ccc-degrees outlines levels of adoption of confidential computing, emphasising the importance of [attestation](#sec-cc-attestation) in verifying that a TEE is trustworthy, and further the integration of attestation into workload-level logic.

In order to effectively use confidential computing, a suitable policy, which attestation reports will be measured against, must be developed _and_ maintained.
While it would be possible for attestation reports to be inspected manually, it is more likely that a {term}`relying party` will establish infrastructure to handle reports.
Furthermore, processes to react to reports, for example excluding non-compliant TEEs from running sensitive workloads, must be implemented.
It would be best for these processes to be automated, which adds to the complexity of the attestation infrastructure.
This aspect could become a significant part of the task of incorporating TEEs into a {term}`TRE implementation`, especially if attestation is conducted on a per-job basis and includes verification of the workload itself [@ccc-degrees].

A number of projects are beginning to build open source tools for managing TEEs ([VirTEE](https://virtee.io/), [Islet](https://github.com/islet-project/)) handling attestation ([VERASION](https://github.com/veraison)) and abstracting TEEs for containers and applications ([dstack](https://dstack.org/), [Ernax](https://enarx.dev/)).
As these tools mature the use of TEEs will become easier and more accessible.

[^performative-security]: Using TEEs while ignoring attestation reports can therefore be seen as "performative security", like taking backup snapshots without verifying their integrity or testing recovery.

### Ongoing management and support

Cost of the upkeep associated with maintaining an applying an attestation policy.
The enforcement of policy can be automated, however, there will unavoidably be a cost in reviewing the policy and keeping it up to date as new hardware is released and if vulnerabilities are discovered in older TEE implementations.

### Vendor lock-in and portability

Introducing TEE support may involve building on vendor-specific hardware, services and APIs.
In these cases, the benefits of TEEs must be balanced against the long-term ability to migrate a TRE instance and the short-term potential for other organisations to deploy your TRE in their own context.
Appetites for lock-in may greatly differ; some organisations may have longstanding good relationships with a vendor, or may have no ambition for their TRE to be deployed by others.

Cloud platforms may provide a more generic interface to deploying CVMs.
However, that comes at the cost of significant buy-in with the cloud platform as a whole.
Some open source projects focus on abstracting TEE management, such as [Open Enclave SDK](https://openenclave.io/sdk/), providing a generic interface to multiple TEE implementations.
As these mature they may make migration between vendors or using multiple vendors in a single system easier.

Furthermore, attestation reports will contain vendor-specific fields, related to that vendor's suite of confidential computing capabilities.
Therefore, incorporating new hardware (in particular hardware from a new vendor) or migrating to different hardware will likely require updating attestation policy.

## Discussion

% TEE summary
To understand whether TEEs add value to a TRE, it is critical to remember that TEEs protect data in use by cryptographically isolating workloads.
This is protection against privileged users on the host, and sophisticated attacks aimed at the OS, hypervisor or BIOS.
Only in situations where attack from the host is a viable risk do TEEs provide a substantial benefit.

% Trust in host
As the administrators of TREs and the infrastructure they run on have, in principle, access to sensitive data it is common to use policy and training to reduce the risk of unauthorised data access or disclosure.
It is common to have a high level of trust in the {term}`infrastructure operator`, in which case a TEE will not significantly reduce risk.
In these cases, while the use of TEEs would reduce the attack surface and make a TRE more secure, the cost and added complexity may not be justified.
Effort would be better spent addressing the more likely risks before considering TEEs.

Beyond the effective and appropriate use of TEEs within a {term}`TRE implementation` there are a number of [key practical considerations](#sec-considerations).
Most importantly, TEEs are not a simple "drop-in" solution and require investment in [hardware](#sec-considerations-hardware), [configuration](#sec-considerations-configuration) and building the [supporting infrastructure to handle attestation reports](#sec-considerations-infra).
Furthermore, support for TEEs is far from universal and requires recent hardware and modern BIOS and OS.
For many organisations this will mean purchasing and provisioning new systems.
Therefore, while TEEs can, in some situations, significantly improve TRE security it is possible that time spent on integrating TEEs may be wasted as access to supported systems will be low.

The situations where we recommend the use of TEEs to enhance TRE security are,

1. [Strong isolation from the host is needed](#sec-recommendations-isolation)
   1. Third party system (TRE hosted by another org, public cloud)
   1. Multi-use system (private cloud _etc._)
1. [Low-trust devices](#sec-recommendations-lowtrust) or bring-your-own-compute (TREs on laptops, mesh TREs)
1. [Extreme data sensitivity](#sec-recommendations-sensitive)

% Common
Perhaps the most common instance where a TEE would add value to a TRE is when the {term}`TRE operator` and {term}`infrastructure operator` are different parties.
This could be a TRE hosted on a public cloud, a TRE hosted by a third-party contractor, or a satellite TRE.
Additionally, where highly sensitive data must be used, TEEs can play an important role in protecting data from motivated attackers who are able to exploit vulnerabilities in the {term}`TRE infrastructure`.

% New models of TRE and trusted research
More excitingly, TEEs could open new models of TRE and new approaches to trusted research where untrusted devices are targeted.
Without confidential computing, we need to ensure a host system is trustworthy and secure to be used.
However, as trust in a TEE does not depend on trust in the host it is possible, in principle, to run trusted workloads securely on compromised and insecure devices.

The use of TEEs could then widen access to TREs and make deploying TREs easier.
This idea is already being explored by the [ManaTEE project](https://manatee-project.github.io/manatee/).
It would also be possible to distribute trusted research to the public,
dependent on confidential computing capability becoming common on consumer devices.
This could be applied to problems which scale well to large numbers of workers,
or to participants contributing to research running analysis locally their own data.
