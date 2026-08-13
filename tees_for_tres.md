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
  - Enhance network security(?)
  - Protect against other users of the _same_ TEE
- Social engineering aspect
  - Protects against "rogue admin"
  - Does not protect against "rogue user"

:::{important} Key point
TEEs provide very strong isolation between the TEE and the host.
Memory encryption, remapping and paging means that other processes on the same host cannot read TEE memory in plain text.
This includes privileged users on the host OS or hypervisor.
Furthermore, some TEE implementations may protect against physical, hardware-based attacks such as reading memory directly (bypassing the CPU), or malicious code running early in the boot process.
:::

:::{important} Key point
Significant use case
Isolation from hardware operator.
For example a datacentre, cloud provider.
:::

:::{important} Key point
TEEs do not provide protection from processes in the TEE.
Therefore, all software in the TEE must be trusted and forms part of the TCB
:::

### Requirements

### Hardware

- Cost of using TEEs
  - Hardware requirements
  - System configuration
  - Processes for handling attestation

:::{important} Key point
Although confidential computing is not new, it has not reached a maturity where all or most devices can be used to provision TEEs.
For CVM solutions, only the most recent, perhaps two or three generations, of devices will have support.
Furthermore, as the technology is still evolving, older CPU generations may lack the features, and enhanced security, or more modern processors.
TEE support is often only available in datacentre CPUs and not present on consumer devices.
Therefore, deploying a new system or upgrading an existing system could come at considerable expense.
:::

### Processes

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

This is precisely the scope (?) of TEEs.
For many TRE scenarios, isolation from the host is not a strong benefit.
For example, where the TRE operator and infrastructure operator are the same organisation and the TRE infrastructure is dedicated (that is, it isn't used for non-sensitive work).
In contrast to this, when the TRE is hosted on infrastructure _not_ controlled by the TRE operator ensuring this isolation is important,

- To minimise attack surface (other users on the host)
- To prevent breach of confidentiality if data is accessed by system administrators

How important this is, will depend on the trust in the infrastructure operator and the guarantees they provide.
For example, large cloud providers make strong statements about their ability to isolate tenant from each other, and from the cloud provider's staff.
Scenarios where TEEs may be a more appropriate solution are,

- TREs hosted on a private cloud alongside non-TRE workloads
- Satellite TREs using a shared responsibility model (like FRIDGE deployed on HPC)
- Federated workflows where ephemeral TREs are deployed within another TRE

Another important scenario is for lower-trust, end-user devices or "bring your own compute".
As a TEE does not depend on trust in the host operating system or software, you can use a TEE to run trusted computation on an untrusted device.
For example, individual laptops or institutional servers which are not designed for secure, multi-tenant use.
This is perhaps closer to the applications of confidential computing outside of trusted research, where a key challenge is running a process on an untrusted machine while ensuring its integrity and confidentiality.

In the extreme, widespread support of TEEs could allow distributed (like [Folding@Home](https://foldingathome.org/) or SETI@Home) trusted research across untrusted devices.

### 2. When strong isolation between jobs is required

 By adopting workload integration of TEEs [@ccc-degrees] and provisioning TEEs per task or job, each workload can be strongly isolated from each other.
 This would be beneficial in TREs where multiple users/projects share the same host or are able to dispatch work to the same runners (like HPC attached to a TRE).
 For example, an organisation curating multiple sensitive data sets, where access to data is managed on a per-person or per-project basis.
 In such situations, building a TRE for each data set would be expensive and make the legitimate combination of data sets difficult.
 However, it is still important to ensure that data is only accessed by the researchers or teams of researchers who are permitted to use it.

### 3. When highly sensitive data must be used

   Other methods to reduce the risk (such as pseudononymisation, data minimisation or the use of synthetic or dummy data) are not possible
   Could be, for example with personal data when disclosure of the data would present a risk to the health or life of subjects or highly sensitive, classified material

<!-- - TEEs add value to TREs (and should be used?) when -->
<!--   - Strong isolation is required from the host -->
<!--     - Here an _entire_ TRE can be hosted in one CVM -->
<!--     - For example shared responsibility -->
<!--     - Satellite TREs -->
<!--     - TREs hosted on private cloud, alongside non-sensitive work -->
<!--     - Federated analysis on TREs -->
<!--     - Non-trusted devices or BYOC -->
<!--   - Strong isolation required between jobs -->
<!--     - For example a multi-user (multi-role might be more descriptive) system -->
<!--     - One TEE per job -->
<!--     - Compute shared between multiple TREs/projects -->
<!--     - Data access is managed by a users' groups (and so attacks between jobs could break confidentiality) -->
<!--     - Indeed, TEEs could enable more efficient use of resources for multi-role TREs -->
<!--   - When the data sensitivity justifies stricter security -->
<!--     - Data which presents a risk to health or life -->

### Cost and value

The above recommendations outline the situations when TEEs add value to a TRE, in terms of the TREs design and usage.
These are based on the discussion of [principles for optimising security](#sec-sec-principles).
However, the decision of whether to use TEEs must also include a consideration of the costs involved, so that an assessment of the net benefit can be made.

Aspects of the costs (financial and otherwise) of using TEEs are [discussed above](#sec-sec-cost).
For the integration of TEEs to TREs we highlight the following costs,

#### Capital expense

For example, investment required in new hardware with confidential computing support.

#### Implementation

The implementation costs can be further split and will likely vary largely depending the TRE design,

- Software development, for example updating IAC to use TEEs, splitting of trusted and non-trusted code (to mimise what is run on a TEE)
- Migrating services to TEEs (@ccc-degrees level 1)
- Developing an attestation policy (@ccc-degrees level 2)
- Building supporting infrastructure, such as "business logic" for involving attestation reports (@ccc-degrees level 3)

#### Ongoing management

Cost of the upkeep associated with maintaining an applying an attestation policy.
The enforcement of policy can be automated, however, there will unavoidably be a cost in reviewing the policy and keeping it up to date as new hardware is released and if vulnerabilities are discovered in older TEE implementations.

#### Vendor lock in and portability

Introducing TEE support may involve building on vendor-specific hardware, services and APIs.
In these cases, the benefits of TEEs must be balanced against the long-term ability to migrate a TRE instance and the short-term potential for other organisations to deploy your TRE in their own context.
Appetites for lock-in may greatly differ; some organisations may have longstanding good relationships with a vendor, or may have no ambition for their TRE to be deployed by others.
Cloud platforms can provide more generic interface to deploying CVMs.
However, that comes at the cost of significant buy-in with the cloud platform as a whole.

Furthermore, attestation reports will contain vendor-specific fields, related to that vendor's suite of confidential computing capabilities.
Therefore, incorporating new hardware (in particular hardware from a new vendor) or migrating to different hardware will likely require updating attestation policy.
