---
title: "Case Study: FRIDGE"
abstract: |
  Considering the integration of TEEs to TREs in the context of the FRIDGE satellite TRE.
authors:
  - name: Jim Madge
    orcid: 0000-0001-6044-164X
  - name: Matt Craddock
    orcid:
license: CC-BY-4.0
keywords:
  - confidential computing
  - trusted execution environments
  - trusted research
  - trusted research environments
  - FRIDGE
---

## Introduction

[FRIDGE](https://alan-turing-institute.github.io/fridge/) is a project enabling the integration of external HPC with a TRE in a secure manner.
It achieves this by deploying a minimal, satellite TRE, capable of orchestrating jobs sent from the "Home TRE".
The satellite is contained within a secure enclave, called the [TRE Tenancy](https://alan-turing-institute.github.io/fridge/introduction/#key-concepts).
The TRE Tenancy is deployed and managed by the host, while the satellite TRE (and everything else inside the tenancy) are controlled by the TRE operator.
This separation of responsibilities is called the [Shared Responsibility Model](https://alan-turing-institute.github.io/fridge/shared-responsibility/)
These two aspects underlie the core principles of FRIDGE,

- Establish a trusted environment on a third party system for trusted research
- Retain governance of the existing TRE
- Exclude the hosting organisation from TRE operations, such as user management, governance or disclosure control

Although the approach of FRIDGE was to minimise the responsibility and involvement of the hosting organisation, they still play a critical role in securing the satellite TRE.
A safe FRIDGE deployment requires the hosting organisation to create a secure TRE Tenancy.
Therefore, the hosting organisation must be trusted to uphold their part of the Shared Responsibility Model
Even when there is a high level of trust,
it is still technically feasible for the hosting organisation to make unauthorised access to the satellite TRE or sensitive data,
whether maliciously or accidentally.

The reliance on the hosting organisation, and potential for interference from the hosting organisation means FRIDGE stands to gain from the inclusion of TEEs
After establishing trust with a TEE, through attestation,
the confidentiality of FRIDGE jobs running on that TEE would be guaranteed without reliance on the hosting organisation, its staff or infrastructure.
Instead, as with all TEEs, the root of trust lies with the CPU (and other trusted hardware) vendor.

This article will consider ways in which the existing FRIDGE archetype could be adapted to incorporate some level of confidential computing.
The options will be assessed with assessment of,

- Security benefit
- Shortcomings (say, size of TCB)
- Complexity of migration
- Readiness
