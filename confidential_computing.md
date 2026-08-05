---
title: Confidential Computing
abstract: |
  A brief summary of confidential computing and trusted execution environments
authors:
  - name: Jim Madge
    orcid: 0000-0001-6044-164X
license: CC-BY-4.0
keywords:
    - confidential computing
    - trusted execution environments
---

## Confidential Computing

Confidential computing describes a set of technologies that protect data while it is in use.
This is distinct from protecting data at rest, written to a storage device, or in transit, while being sent over a network.
We could consider both of those as solved problems, with public/private key encryption, secure protocols and modern filesystems.
Confidential computing is achieved by creating a hardware-based, attestable TEE.
Attestation, proves that a TEE is correctly configured and has not been tampered with.
Data in use, and code being executing in a TEE is encrypted and may not be read or modified by other processes on the same computer.

## Trusted Execution Environments

A TEE is a reserved portion of hardware (CPU, memory and possibly GPU) that is cryptographically segregated from the rest of the system (the host, and any other TEEs).
The contents of a TEE are unable to be read, or modified by any processes outside of the TEE, including those running on the same host.
TEEs are the enabling technology for confidential computing [@ccc-terminology].
They operate by selectively encrypting sections of memory belonging to TEEs with enclave-unique keys.
This is done by a trusted {term}`secure processor` which ensure encrypted memory can only be read as plain text by the correct TEE.
Implementations vary as to how this is achieved but generally involve some kind of memory address virtualisation or remapping in addition to encryption.
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
An overview of this is shown in [](#fig-tcb-and-rot).

:::{important} Trust
You must have ultimate and explicit trust is in the CPU vendor.
You must trust that,

1. they have produced a CPU capable of provisioning TEEs
1. they have produced a CPU capable of generating accurate evidence
1. their endorsement of the CPUs genuine status is correct

The hardware root of trust is the CPU, which produces evidence.
Evidence is verified by the CPU vendor, or entities on a chain of trust leading back to the vendor.
:::

::::{figure}
:label: fig-tcb-and-rot
:::{image} https://mermaid.ink/img/pako:eNp9lE1zmzAQhv8Ko1yph2-Mekps59ZpZ5JeWjodBQmbCSBGEo5dj_97VxLgmMblgvbRfrxaLZxQwSlDGL3UvHjNW8cpeN03rXRCbRiKt4L33W_FGA40vPgMpuSleiOCWYsSRfSKtVS_IOxnjp43G4e01HlePeTo1zz1jks1z-1bc3fsmNhXkgtrc7VjAhLqkKmw61BR7ZmQQ24Ig20uR7-vT8PGoElD2OtbJXqpGDUAKjcdb1mr5AcSBb-lsKxEo0VAwnFpjtpUheC6u5OoouvBafXt-0T2IIgLCwfjWimU1Ycggpq8gnPl8NIxuo1n3hY1kXLNSqcmL6wGOXWN7x7N40ol-Cv79FZRtcNed_g8uOtbsf4jMB24IlD6CugSphf_KRF1hwHgu8w8U8VpiOzqGpvqH3Ct4T3WGsY7syqi9f1jdH_7oOOIjGEj11M6Z3A_czTd6Izbu5pRre4yU1ZfmKabh-VtfXZQL2ETn-b-3z3zDbzHyEUNEw2pKHzJJz03OQKfBkYPw5KykvQ1zEvensGV9Io_HdsCYUjAXAT93e4QLkktweo7aA1bV2QrSDPRjrQ_OG_GEEYrxcUX--swfxDjgvAJHRD2s3iRRoHv-4GXeHGUuuiIcJAGCy-L_WUUBomf-tnZRX9MTm8RZ8sk8cIgjoIwTLLERVuhDzNUF9BtJlYcjgzZ4_NfPl2j3w?type=png
:::
% :::{mermaid}
% block
%   columns 3
%   block:group_tee:2
%     columns 2
%     software
%     data
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
% class cpu trusted
% class firmware trusted
% class vendor trusted
% classDef untrusted fill:#377EB8,stroke-width:0px;
% class hostos untrusted
% class hypervisor untrusted
% class other untrusted
% :::
A block diagram showing an example of a TEE implementation, showing the TCB, the untrusted host resources, and the hardware root of trust.
::::

(sec-cc-attestation)=
## Attestation

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

### Attestation Evidence

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

### Remote Attestation

It is not feasible for most organisations to review attestation evidence themselves, due to the complex and technical nature of the evidence [@ccc-attestation].
Therefore, it is expected that most TEE users will rely on a {term}`verifier` to assess the evidence and obtain endorsements.
This is outlined in @rfc9334.
In these cases the {term}`relying party owner` must trust the {term}`verifier`.
[RATS trust model](https://www.rfc-editor.org/info/rfc9334/#name-relying-party)

Remote attestation is a process that uses a third party (_i.e._ neither the TEE, nor the part requesting verification of the TEE) to assess the TEE.
This is defined formally in IETF's  request for comments 9934 @rfc9334.
The RFC sets out a number of roles, which are paraphrased in [](#sec-rats).

:::{important} Attestation result
It is important to understand that, although the {term}`verifier` will have a policy to verify the validity of evidence, the {term}`verifier` _does not_ make a decision on behalf of the {term}`relying party` as to whether they should trust a TEE.
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
  vo[Verifier Owner]
  rvp[Reference Value Provider]
  Endorser
  rp[Relying Party]
  rpo[Relying Party Owner]
  Attestor --> |Evidence| Verifier
  Endorser --> |Endorsements| Verifier
  rvp --> |Reference Values| Verifier
  vo --> |Policy for Evidence| Verifier
  Verifier --> |Attestation Results| rp
  rpo --> |Policy for Results| rp
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
