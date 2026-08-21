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

Confidential computing describes a set of technologies that protect data while it is in use [@ccc-confidential-computing].
This is distinct from protecting data at rest, written to a storage device, or in transit, while being sent over a network.
We could consider both of those as solved problems, with public/private key encryption, secure protocols and modern filesystems.
Confidential computing is achieved by creating a hardware-based, attestable TEE.
Attestation, proves that a TEE is correctly configured and has not been tampered with.
Data in use, and code being executing in a TEE is encrypted and may not be read or modified by other processes on the same computer.

(sec-cc-tees)=
## Trusted Execution Environments

A TEE is a reserved portion of hardware (CPU, memory and possibly GPU) that is cryptographically segregated from the rest of the system (the host, and any other TEEs).
The contents of a TEE are unable to be read, or modified by any processes outside of the TEE, including those running on the same host.
TEEs are the enabling technology for confidential computing [@ccc-terminology].
They operate by selectively encrypting sections of memory belonging to TEEs with enclave-unique keys, which ensure encrypted memory can only be read as plain text by the correct TEE.
Implementations vary as to how this is achieved but generally involve some kind of memory address virtualisation or remapping in addition to encryption.
A TEE can prove its state (and hence that it has been configured correctly) though an [attestation](#sec-cc-attestation) process, so its users can make an informed decision whether to trust it or not.

Both memory encryption and gathering evidence for attestation are performed by a trusted {term}`secure processor`.
This is a section of the CPU dedicated to confidential computing functions and enforcing the security of TEEs.
It provides an API for managing the TEE platform and enclaves for example, reporting firmware versions, creating an enclave, generating attestation evidence.
However, it cannot be arbitrarily controlled by the host to bypass enclave isolation for instance, decrypting TEE memory outside of the enclave or returning enclave encryption keys.
The secure processor therefore plays a critical role in the operation of a TEE.

:::{important} Trust
You must have ultimate and explicit trust in the CPU vendor.
You must trust that,

1. they have produced a CPU capable of provisioning TEEs
1. they have produced a CPU capable of generating accurate evidence
1. their endorsement of the CPUs genuine status is correct

The hardware root of trust is the CPU, which produces evidence.
Evidence is verified by the CPU vendor, or entities on a chain of trust leading back to the vendor.
:::

(sec-cc-security)=
## TEE security

TEEs provide very strong, cryptographic isolation between the enclave and the host.
Memory encryption and address remapping ensure TEE processes are confidential and their memory cannot be read in plain text outside of the enclave.
This confidentiality includes privileged users on the host OS or hypervisor.

A TEE implementation may also protect against some physical, hardware-based attacks such as reading memory directly (bypassing the CPU), or malicious code running early in the boot process.
Furthermore, as the lifecycle of a TEE and the attestation process do not depend on trust in the host, a TEE remains secure even in the case of the host OS, hypervisor, software, drivers or firmware being compromised.

:::{important} Key message
In summary, TEEs

- Remove the need to trust many software and hardware components of a host, and the operator of that host
- Ensure processes are confidential even when security vulnerabilities are exploited on the host
:::

:::{note} TEE use cases
:class: dropdown
TEE development has in part been driven by commercial use cases where a service wants to keep their process confidential from an untrusted, potentially insecure end-user device.
This could be, for example, a banking application running on a phone, or a video streaming service for copyrighted material.
:::

(tip-tcb)=
:::{tip} Trusted compute base
:class: dropdown
The TCB of a computer system is the set of all components (hardware, software and firmware) that play a role in providing the required security.
A flaw on any member of the TCB could mean the security of the entire system is compromised.

A principle of computer security is to minimise the size of the TCB, in other words, reducing the number of critical component for security.
This reduces the attack surface and simplifies the monitoring and maintenance of security-critical components.
:::

The [TCB](#tip-tcb) of a TEE can vary based on the [design of the TEE](#sec-ccs-cc-archetypes).
However, it will always exclude the host OS and any hypervisor as the TEE is cryptographically segregated from these and they play no role in attestation
The host CPU, and its firmware and microcode, are critical to the security of a TEE, and so are in the [TCB](#tip-tcb).
They form the hardware root of trust.
The CPU and CPU manufacturer are therefore the root of the chain of trust for the entire TEE and ultimately you must trust these organisation and their products.
An overview of the [TCB](#tip-tcb) and hardware root of trust are shown in [](#fig-tcb-and-rot).

::::{figure}
:label: fig-tcb-and-rot
:::{image} https://mermaid.ink/img/pako:eNp9kl1vmzAUhv8Kcm82iUUQvoJ31Sbp7Satu9mYJhebgAo2sk2bLsp_37H5SEibokjxeXw-XtvvAeWCMoTRYy3yp4w7Ti7qruHKCUxgKd5J0bV_NWN4aeApZwiVKPQLkayPKNHErBin5g_KfmfoYbt1Pj2s7z5n6M9l41IofdnZ78PytWXyuVJC9rHQJZPQzpRMY12HyuqZSTX0hjLYFmrM-_Zj2BgUGQh7HdeyU5pRC2By0wrOuFbvSJTimsKiko0RAQ3HpUM4dZoql8Lc7SQqbztIWn__OVcDrY1QIqmtlUJoRxSO1XZ2ZRnPa6LUhhVOTR5ZDYPrGt_c289VWoon9uWlorrEXrv_OqSb2-_zR2DPOiMgYAbMCHvqD0aE7X4A-Ca13zRxMku_mmM7_R1uNJxjo2F8nV5FuLm9D2-vH3Q0w1g2cuPGSwYvcYmmt5tzo-Pkk15JkCTbu9V1Jb35TmUTn7z8ds_6-hwjF-1kRREGwlzUMNkQE6KD8UyGIL8Ba2FYUlaQrtYZyvgRylrCfwnRjJVwqbsS4YLUCqKuhftgm4rsJGkmKsGKTK4FzEd4uQpsE4QPaI-wH4eLcJWkfuQFqRcvIxe9IhzG3iIIvMRfpWnoh158dNE_O9VbRH5sfmEQR2mQxNHxP78DhfA?type=png
:::
A block diagram showing an example of a TEE implementation, showing the TEE, TCB, the untrusted host resources, and the hardware root of trust.
% :::{mermaid}
% block
%   columns 3
%   block:group_tee:2
%     columns 2
%     software
%     data
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
% class cpu trusted
% class firmware trusted
% classDef untrusted fill:#377EB8,stroke-width:0px;
% class hostos untrusted
% class hypervisor untrusted
% class other untrusted
% :::
::::

TEEs don't address vulnerabilities of code or processes _inside_ the TEE.
Malicious software (including a guest OS) inside an enclave may still have access to secrets and be able to exploit these or expose them outside the TEE.

Similarly, TEEs do not protect against users able to interact software inside a TEE, such as an OS or API.
Users may be able to use their privileges, or exploit a vulnerable software inside the TEE, to access or leak sensitive data.
TEEs do not, therefore, rule out the need for security best practices regarding user access and the application running inside the enclave.

:::{important} Trusted software
TEEs do not provide protection from processes in the TEE.
Therefore, all software (including any OS) in the TEE must be trusted, and forms part of the [TCB](#tip-tcb).
This is illustrated in [](#fig-tcb-and-rot).
:::

Advanced side-channel, physical-access attacks have been use to expose TEE secrets.
However, an attack able to read arbitrary memory in plain text has not been demonstrated.
Proven attacks include [tee.fail](https://tee.fail/) and [wiretap.fail](https://wiretap.fail/).

(sec-cc-attestation)=
## Attestation

Attestation is the process by which a TEE proves that it is secure.
This is how trust is established when interacting with a TEE.
Attestation verifies that,

- The TEE has been configured correctly
- The TEE is running on authentic hardware

In some cases, it may also verify that some software component of the TEE is in a known state, _i.e._ it has not been modified or tampered with.

(sec-cc-attestation-evidence)=
### Attestation Evidence

In order to attest the legitimacy of a TEE, evidence about that TEEs state must be assessed.
The evidence consists of a number of claims made by the TEE.
Each claim consists of a name and value.
The claims will vary, depending on a TEEs configuration and hardware, and may include,

- CPU manufacturer, model
- Firmware and microcode versions
- TEE configuration data (such as types of memory encryption and address remapping)
- Hashes of software in a TEE

All claims are collected as an evidence object, such as JWT [@rfc7519] or EAT [@rfc9711].

It is important that evidence is correct so a TEE can be accurately assessed.
The evidence is collected by collected by hardware-based routines[^firmware-microcode] in the {term}`secure processor` making it unfeasible for the host or hypervisor to tamper with the evidence
The evidence object is signed by a private key belonging to the {term}`secure processor`.
Unique, unpredictable, random private keys for each CPU are generated at the time of manufacture.
They are written to the hardware in one-time PROM, and so are immutable.
This signature can be verified by the hardware vendor, which keeps a set of corresponding public keys to use in identity verification challenges.
It is therefore not possible to mimic a legitimate system through virtualisation.
If the signature is verified, and the hardware vendor is trusted, the claims are known to be correct and produced by the CPU.

[^firmware-microcode]: In principle this also includes firmware and microcode, which may be modified after manufacture. The versions and integrity of firmware and microcode should therefore be verified as part of attestation.

### Remote Attestation

It is not feasible for most organisations to review attestation evidence themselves, due to the complex and technical nature of the evidence [@ccc-attestation].
Therefore, it is expected that most TEE users will rely on a {term}`verifier` to assess the evidence and obtain endorsements.
This is defined formally in IETF's  request for comments 9934 [@rfc9334].
In these cases the {term}`relying party owner` must trust the {term}`verifier` and {term}`verifier owner` to process evidence on their behalf.

Remote attestation is a process that uses a third party (_i.e._ neither the TEE, nor the part requesting verification of the TEE) to assess the TEE.
The RFC sets out a number of roles, which are paraphrased in [](#sec-rats).

:::{important} Attestation result
It is important to understand that, although the {term}`verifier` will have a policy to verify the validity of evidence, the {term}`verifier` _does not_ make a decision on behalf of the {term}`relying party` as to whether they should trust a TEE.
It is always the responsibility of the {term}`relying party` to inspect the attestation report and decide what to do following the {term}`relying party owner's <relying party owner>` policy.
:::

The basic workflow of remote attestation, showing how data flows between roles, is shown in [](#fig-attestation).

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

:::{important} Trust
The roles and remote attestation process illustrates where trust lies when using a TEE.
The {term}`attester` does not validate itself or make a statement about its security, it produces evidence.
The {term}`verifier` processes evidence must be trusted to correctly arrive at its conclusions.
Therefore, any {term}`endorsers <endorser>` and {term}`reference value providers <reference value provider>` must also be trusted to provide correct information to the {term}`verifier`.
The RATS trust model is explained in more detail [in RFC 9334](https://www.rfc-editor.org/info/rfc9334/#name-relying-party).
:::

The actual procedure of a remote attestation process may be implemented in different ways.
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
- Withdrawal of service from attestation providers or reliance on third parties for organisations where TEEs in critical.
:::
