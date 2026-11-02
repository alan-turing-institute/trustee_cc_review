---
title: Confidential Computing Experiments
abstract: |
  A report on deploying a confidential computing system on local CPU/GPU hardware hosted by UCL ARC.
authors:
  - name: Tuomas Koskela
    orcid: 0000-0002-5813-6539
  - name: James Legg
license: CC-BY-4.0
keywords:
    - confidential computing
    - trusted execution environments
    - high performance computing
    - cloud computing
    - trusted research
---

## System Overview

We use a system "Airy" hosted in UCL ARC's "Contender" testbed [@arc-contender]. The system has a Intel Xeon 6517P Grantie Rapids CPU with full support for both Intel SGX and TDX security extensions. It has four NVIDIA RTX PRO 6000 Blackwell Server Edition GPUs, connected via a PCIe bridge.

## Hardware Setup

## CVM Setup

## CPU Attestation

The Intel Trustauthority client is an open-source project for attesting TEEs with Intel Trust Authority. It provides an API and a CLI for generating and verifying attestation claims from TDX hardware evidence [@intel-trustauthority-client]. Users must register with the trustauthority service to be able to create API keys.

The CLI client runs on the guest VM, and it needs a way to communicate with the quote generation service running on the host. This is/was discussed in Hardware Setup. The client is configured by writing a JSON file

```json
{
  "trustauthority_url": "https://portal.eu.trustauthority.intel.com",
  "trustauthority_api_url": "https://api.eu.trustauthority.intel.com",
  "trustauthority_api_key": "<api key hash>"
}
```

Once the API keys are generated and the client is configured, it can be used to generate an attestation token with
```bash
trustauthority-cli token -c ~/trustauthority_config.json
```
The token is a JWT token. It is base64 encoded and it contains three fields separated by `.`, header, payload and signature. It's written to stdout by default. The `trustauthority-cli` also provides functionality to verify the token with the trustauthority portal
```bash
trustauthority-cli verify -c ~/trustauthority_config.json -t $TOKEN
```
with the attestation token stored in `$TOKEN`.

Alternatively, the token can be decoded and verified with the FRIDGE verification app (https://github.com/alan-turing-institute/FRIDGE-attestation/tree/main/verification_app). The URL of the issuing certificate authority is written into the signature field of the token. The FRIDGE app will read it and verify the token signature and validity window with the certifier.

The JWT format is widely used and a variety of tools for decoding the token are available, such as https://jwt.io and https://pypi.org/project/jwt/.

### Guest Memory Access from Host

The memory of a non-confidential virtual machine is by default accessible by the host operating system. We have shown that a confidential virtual machine blocks the access by encrypting the guest's memory from the host.

## GPU Attestation

The Nvidia Attestation SDK is an open source project for implementing and validating TEEs on Nvidia hardware. It provides C and Rust APIs and a CLI (`nvattest`) for generating and verifying attestation claims from a Nvidia GPU.



## Example Workloads
