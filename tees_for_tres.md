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

## Security

(sec-sec-principles)=
### Principles

- High level coverage of security concepts
  - Attack Surface
  - Trusted Compute base
  - Types of attack (including social engineering)
- Security principles
  - Minimise attack surface
  - Minimise trusted compute base

:::{important} Key point
The TCB is all of the components of a computer system responsible for enforcing security.
A principle of computer security is to minimise the size of the TCB, to reduce potential routes for attacks and make auditing more managable.
As all software inside a TEE may have access to sensitive data, it must be considered part of the TCB
:::

- Non technical
  - Using policy/process as cover for missing technical controls
  - Training
  - Good data practice, minimising data, psuedonymisation, use of synthetic data
  - When a process is cheaper/more efficient than a technical implementation
  - Security in depth

:::{important} Key point
In TREs, it is appropriate to use non-technical controls to build trust and ensure security.
There are many options available to manage risk and the use of a TRE does not mean that other strategies can be neglected or are no longer needed.
This is especially true in cases where technical solutions are impractical or complex.
A more holistic view of security when handling sensitive data is described by [The Five Safes](https://ukdataservice.ac.uk/help/secure-lab/what-is-the-five-safes-framework/)
:::

(sec-sec-cost)=
### Security cost

- The cost of increased security
  - Deployment effort
  - Upkeep/management effort (increased complexity, specialist skills)
  - Encouraging workarounds or shadow IT
  - Vendor lock-in and portability

- Conclusion
  - Should consider the benefit of additional security against the costs
  - Are there other ways to achieve this?
  - May not be worth it when
    - Cost is very high
    - The benefit is minimal
      - overlap with existing controls, won't reduce attack surface (maybe not the right terms here)
      - Possibility to shift trust in a way that makes TCB more complex

### Cost and Benefit

- Why not _all_ the security _all_ the time

(sec-security)=
## TEE Security

- What does a TEE do?
  - Confidentiality from host, hypervisor
  - Confidentiality from _other_ TEEs
  - Removes the need for trust in host, apart from CPU and firmware
- What a TEE doesn't do
  - Removes the need to trust code inside the TEE
  - Protect against other users of the _same_ TEE
  - Some types of physical-access attack
- Social engineering aspect
  - Protects against "rogue admin"
  - Does not protect against "rogue user"

To assess the value of leveraging confidential computing in a TRE it is necessary to understand specifically what protection TEEs offer (and how those intersect(?) with TREs)

TEEs provide very strong, cryptographic isolation between the enclave and the host.
Memory encryption and remapping or paging ensure that other processes on the same host cannot read TEE memory in plain text.
This includes privileged users on the host OS or hypervisor.
Furthermore, some TEE implementations may protect against physical, hardware-based attacks such as reading memory directly (bypassing the CPU), or malicious code running early in the boot process.

:::{important} Key point
use cases
  Isolation from hardware operator. For example a datacentre, cloud provider.
  Isolation from other workloads. For example a datacentre, cloud provider.
  Untrusted user or device. For example banking app on phone
:::

:::{important} Key point
TEEs do not provide protection from processes in the TEE.
Therefore, all software in the TEE must be trusted and forms part of the TCB
Guest is trusted
:::

## TEE Implementation/Requirements

- more brief detail, or skip entirely in favour of considerations section
  - hardware
  - configuration
  - infrastructure/processes

(sec-tre-models)=
## TRE Models

There is a tremendous diversity in the implementation of TREs.
It is still common for organisations to build or commission their own TRE and they are often designed for the needs of one organisation.
As such, despite the emergence of archetypes and common tools, there is no standard TRE implementation or architecture.
This makes deciding where a TEE could add value to a TRE more difficult, requiring consideration of a particular TRE's design and use.
However, we can consider the "dimensions" of TRE design to make more general recommendations about which styles of TRE stand most to gain from the adoption of confidential computing.

- Dimensions
  - Depending how TEEs are used…
    - one TEE for whole TRE (no user-user isolation)
    - TEE for each user (user-user isolation)
    - TEE for each job (workload-level isolation)
  - How is data accessed
    - All users have access to data
    - RBAC to assign an identity all of "their" data sets (possibility to mix datasets against governance rules)
    - User/project matrix. Each person has N accounts for their N projects (already strongly isolates datasets which should not be mixed)
  - Hosting
    - How much trust in host
    - Same organisation or split across organisations
- Examples/case studies
  - SAIL/Scottish NHS
  - ONS?
  - OpenSAFELY
  - Aridhia

## Roles

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
: An person authorised to use a {term}`TRE`.
:::

## Entities

::: {glossary}
TRE Project
: A piece of sensitive work with a single set of governance outlining its rules
  This could be a research project addressing a particular question, or the curation of an important data set

TRE
: The environment used by a project, with rules defined by a single governance …
  A single deployment of a {term}`TRE implementation`

TRE Implementation
: The software/infrastructure that enables …

TRE Infrastructure
: The underlying platform on which a {term}`TRE` is build.
  For example a private, public cloud, a single workstation.
:::

(sec-recommendations)=
## Recommendations

By considering the [design of TREs](#sec-tre-models) and [the scope of TEEs protection](#sec-security) we have arrived at the following situations where TEEs as substantial value to TREs and should be considered.

### 1. When strong isolation from the host is required

Providing a way for compute to be run on an untrusted host, or host with untrusted users, is the central goal of confidential computing.
This aligns well with many existing TRE scenarios, particularly where TREs are deployed to systems not entirely dedicated to trusted research (for example, local HPC or private cloud), or where a TRE is deployed to a third-party's system.
Only when the same party holds the {term}`TRE operator` and {term}`infrastructure operator` roles _and_ the {term}`TRE infrastructure` is dedicated
(that is, it isn't used for non-sensitive work) is isolation from the host not a strong benefit.


#### 1.a. TRE operator and infrastructure operator are different parties

When the {term}`TRE operator` and {term}`infrastructure operator` roles are not held by the same party, the {term}`TRE infrastructure` is _not_ controlled by the {term}`TRE operator`.
The {term}`infrastructure operator` will have a high degree of privilege and access to the {term}`TRE infrastructure`, which potentially allows them to observe sensitive processes in breach of TRE governance.
The cryptographic isolation of a TRE from the {term}`TRE infrastructure` using a TEE ensures that the {term}`infrastructure operator` is not able to access data in use, whether maliciously or accidentically.

Precisely how important this is will depend on the trust in the infrastructure operator and the guarantees they provide.
For example, large cloud providers make strong statements about their ability to isolate tenant from each other, and from the cloud provider's staff.
However, even in cases where there is absolute trust in the {term}`infrastructure operator` TREs are still vulnerable to attack from a compromised {term}`TRE infrastructure`.
Other scenarios where TEEs provide valuable isolation to TREs more appropriate solution are,

- Satellite TREs using a shared responsibility model (such as [FRIDGE](https://alan-turing-institute.github.io/fridge/) deployed on HPC)
- Federated research where one {term}`TRE operator` runs a workload or deploys an ephemeral TRE within another TRE.

#### 1.b. TRE infrastructure is not exclusive

When the {term}`TRE infrastructure`is not used exclusively for trusted research, the risk of non-authorised administrators accessing sensitive data (whether accidentally or maliciously) is raised.
In this scenario, different rules and processes for the administration of sensitive and non-sensitive environments could lead to errors.
An example of this would be an HPC system which provides a service for both sensitive and non-sensitive workloads.
This model could be particularly effective on a private cloud, a {term}`TRE project` would be allocated a CVM while non-sensitive work is conducted with conventional VMs.

#### 1.c. Low trust devices

Another possible scenario is low-trust, end-user devices or "bring your own compute".
As a TEE does not depend on trust in the host operating system or software, you can use a TEE to run trusted computation on an untrusted device.
For example, individual laptops or institutional servers which are not designed for secure, multi-tenant use.[^commericaluse]
In the extreme, widespread support of TEEs could allow distributed trusted research across untrusted devices, which would currently be far too risky to consider.
Perhaps this could scale to large pools of workers similarly to how non-sensitive distributed research has been conducted by [Folding@Home](https://foldingathome.org/) or SETI@Home.

[^commericaluse]: This is perhaps closer to the applications of confidential computing outside of trusted research, where a key challenge is running a process on an untrusted machine while ensuring its integrity and confidentiality.

:::{important} Isolating jobs or users
TEEs are _not_ necessary for isolating individual jobs or {term}`TRE projects <TRE project>` from each other.
This may be important in a {term}`TRE implementation` which supports multiple {term}`TRE projects <TRE project>` using the same {term}`TRE infrastructure`.
In these cases, isolation can be introduced by using VMs.
Since no {term}`TRE user` has no access to the host or hypervisor, the attacks which TEEs protect against are not viable.
:::

### 2. When highly sensitive data must be used

When dealing with highly-sensitive data, computer security is not the only option for protecting against unauthorised disclosure.
Instead, in line with the Five Safes framework [@five-safes], a more holistic approach to security should be taken and the data should be modified to reduce disclosure risk if possible (for example pseudononymisation, data minimisation or the use of synthetic or dummy data).
When highly-sensitive data must be used, however, it is appropriate to reduce risk in other dimensions.
In these cases, the use of TEEs may be beneficial.
It would offer most benefit when dealing with data which would encourage high-motivated attackers to launch sophisticated attacks at the infrastructure level.
For example, compromising the host OS, BIOS or a social engineering attack targeting the {term}`infrastructure operator`.

## Considerations

The [above recommendations](#sec-recommendations) outline the situations when TEEs add value to a TRE, in terms of the TREs design and usage.
These are based on the discussion of [principles for optimising security](#sec-sec-principles).
However, the decision of whether to use TEEs must also include a consideration of the costs involved, so that an assessment of the net benefit can be made.

Aspects of the costs (financial and otherwise) of using TEEs are [discussed above](#sec-sec-cost).
For the integration of TEEs to TREs we highlight the following costs,

### Hardware and Provisioning

Although confidential computing is not new, it has not reached a maturity where all or most devices can be used to provision TEEs.
For CVM solutions, only the most recent, perhaps two or three generations, of devices will have support.
Furthermore, as the technology is still evolving, older CPU generations may lack the features, and enhanced security, or more modern processors.
TEE support is often only available in datacentre CPUs and not present on consumer devices.
Therefore, provisioning compatible hardware in a new system or existing system could come at considerable expense.

In addition to capital investment in hardware, an organisation implementing confidential computing on-premises will need to perform the necessary configuration and setup.
This may include,

- BIOS (UEFI) configuration
- Installing a compatible Host OS
- Host OS configuration
- Guest OS configuration (for example, building a minimal guest image with necessary tools)
- Configuring other trusted hardware, such as GPUs or NICs

Example setup guides from [AMD](https://docs.amd.com/v/u/en-US/58207-using-sev-with-amd-epyc-processors) and [Intel](https://cc-enabling.trustedservices.intel.com/intel-tdx-enabling-guide/01/introduction/)

Both costs can be avoided by using a third-party, such as a cloud computing provider, to provide TEEs as a service
A reduction in upfront expense … in exchange for (likely) larger operational expenses

### Processes

Using TEEs also requires supporting infrastructure for supporting attestation and integrating it into workflows …

The implementation costs can be further split and will likely vary largely depending the TRE design,

- Software development, for example updating IAC to use TEEs, splitting of trusted and non-trusted code (to mimise what is run on a TEE)
- Migrating services to TEEs (@ccc-degrees level 1)
- Developing an attestation policy (@ccc-degrees level 2)
- Building supporting infrastructure, such as "business logic" for involving attestation reports (@ccc-degrees level 3)

CCC white paper on degrees of confidential computing [@ccc-degrees].
Simply using confidential guests, without verifying their state should give a high level of confidentiality from the host.
However, by skipping the attestation, systems which interact with the TEE (and may rely on it being secure and confidential) cannot decide whether to trust the TEE.
Comparing the state of a TEE against an organisational policy is a key part of establishing trust in a TEE.
Skipping this, confidential and non-confidential VMs are treated equally.
Can think of this like taking backups without verifying their integrity or testing recovery.

The two further levels are,

2. verifying the state of the infrastructure (infrastructure level attestation)
3. workload level

In order to effectively use confidential computing, an organisation must develop _and_ maintain a suitable policy which attestation reports will be measured against.
There must be processes in place (whether automated or manual) to exclude non-compliant TEEs from handling sensitive workloads.

- Most likely a remote attestation service ({term}`verifier` role)


### Capital expense

For example, investment required in new hardware with confidential computing support.

### Implementation cost

The implementation costs can be further split and will likely vary largely depending the TRE design,

- Software development, for example updating IAC to use TEEs, splitting of trusted and non-trusted code (to mimise what is run on a TEE)
- Migrating services to TEEs (@ccc-degrees level 1)
- Developing an attestation policy (@ccc-degrees level 2)
- Building supporting infrastructure, such as "business logic" for involving attestation reports (@ccc-degrees level 3)

### Ongoing management

Cost of the upkeep associated with maintaining an applying an attestation policy.
The enforcement of policy can be automated, however, there will unavoidably be a cost in reviewing the policy and keeping it up to date as new hardware is released and if vulnerabilities are discovered in older TEE implementations.

### Vendor lock in and portability

Introducing TEE support may involve building on vendor-specific hardware, services and APIs.
In these cases, the benefits of TEEs must be balanced against the long-term ability to migrate a TRE instance and the short-term potential for other organisations to deploy your TRE in their own context.
Appetites for lock-in may greatly differ; some organisations may have longstanding good relationships with a vendor, or may have no ambition for their TRE to be deployed by others.
Cloud platforms can provide more generic interface to deploying CVMs.
However, that comes at the cost of significant buy-in with the cloud platform as a whole.

Furthermore, attestation reports will contain vendor-specific fields, related to that vendor's suite of confidential computing capabilities.
Therefore, incorporating new hardware (in particular hardware from a new vendor) or migrating to different hardware will likely require updating attestation policy.

## Conclusion

% TEE summary
To understand whether TEEs add value to a TRE, it is critical to remember that TEEs protect data in use by cryptographically isolating workloads.
This is protection against privileged users on the host, and sophisticated attacks aimed at the OS, hypervisor, BIOS.
Only in situations where attack from the host is a viable risk do TEEs provide a substantial benefit.

% Trust in host
As the administrators of TREs and the infrastructure they run on have, in principle, access to sensitive data it is common to use policy and training to reduce the risk of unauthorised data access or disclosure.
It is common to have a high level of trust in the {term}`infrastructure operator`, in which case a TEE will not significantly reduce risk.
In these cases, while the use of TEEs would reduce the attack surface and make a TRE more secure, the cost and added complexity may not be justified.
Addressing the more likely risks first …

Perhaps the most common instance where a TEE would add value to a TRE is when the {term}`TRE operator` and {term}`infrastructure operator` are different parties.
This could be a TRE hosted on a public cloud, a TRE hosted by a third-party contractor, or a satellite TRE.

% New models of TRE and trusted research
More excitingly, TEEs could open new models of TRE and new approaches to trusted research where untrusted devices are targeted.
Without confidential computing, we need to ensure a host system is trustworthy and secure to be used.
However, as trust in a TEE does not depend on trust in the host it is possible, in principle, to run trusted workloads on …
The use of TEEs could then widen access to TREs and make deploying TREs easier.
This idea is already being explored by the [ManaTEE project](https://manatee-project.github.io/manatee/).
It would also be possible to distribute trusted research to end user devices, which could be applied to problems which scale well to large numbers of workers, or to participants contributing to research by running … locally their own data
This would be dependent on confidential computing capability becoming common on consumer devices; currently in computer hardware it is exclusive to datacentre/enterprise.

In summary, the situations when we recommend the use of TEEs to enhance TRE security are,

1. Strong isolation from the host
   1. Third party system (TRE hosted by another org, public cloud)
   1. Multi-use system (private cloud _etc._)
   1. Bring-your-own-compute (TREs on laptops, mesh TREs)
1. Extreme data sensitivity

Beyond the effective and appropriate use of TEEs within a {term}`TRE implementation` there are a number of key practical considerations.

% Readiness of hardware/adoption
Although CC technology is not new, it is not yet a standard feature across architectures or vendors suite of offerings.
Currently, you can expect to only find CC support in the latest few generations of server/datacentre CPUs.
Integration of devices into TEEs is not generally solved, although the latest generations of GPUs from Nvidia and AMD can be incorporated into a TEE.
For many, it may therefore not be possible to build on TEEs without a large investment in new hardware.
And so, while TEEs can, in some situations, significantly improve TRE security it is possible that time spent on integrating TEEs may be wasted as access to supported systems will be low.

% Key considerations
Furthermore, making the most of TEEs is not simply a case of enabling secure virtualisation.
@ccc-degrees outlines levels of adoption of confidential computing, emphasising the importance of [attestation](#sec-cc-attestation) in verifying that a TEE is trustworthy, and further the integration of attestation into workload-level logic.
Even just handling a single attestation report requires the {term}`relying party` to set up related infrastructure such as an attestation policy and a mechanism for reacting to a report.
This aspect should not be ignored, and could become a significant part of the task of incorporating TEEs into a {term}`TRE implementation`, especially when integrated to a high degress such as removing not-compliant nodes from a pool or requiring passing attestation before each job is launched.
