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

## Overview

The lifecycle of a QEMU/KVM virtual machine configured with NVIDIA GPU passthrough was managed by three scripts on the host:

- **`launch-gpuAB-vm.sh`** — Launch a TDX (Trusted Domain Extension) protected VM with GPU passthrough
- **`launch-notdx-gpuAB-vm.sh`** — Launch a standard (non-TDX) VM with GPU passthrough
- **`shutdown-gpuAB-vm.sh`** — Gracefully power down the running VM

### VM Architecture

Both VMs are built on:
- **QEMU/KVM** hypervisor with KVM acceleration
- **q35** chipset (Intel platform)
- **4 vCPUs** and **64GB RAM**
- **OVMF/UEFI firmware** (Intel BIOS)
- **Virtio block storage** (ubuntu-gpuA.img image)
- **Cloud-init** seed image (td-guest-seed.img) for guest initialization
- **Virtio network** bridging to host (virbr0)

### Key Differences: TDX vs Non-TDX

#### TDX VM (`launch-gpuAB-vm.sh`)

| Feature | Configuration |
|---------|---|
| **Firmware** | Intel TDX BIOS (`OVMF.inteltdx.fd`) |
| **Memory Backend** | TDX-aware memory backend (`memory-backend-ram`) |
| **Machine Type** | `q35` with `confidential-guest-support=tdx` |
| **Encryption** | Full guest memory encryption via TDX |
| **Quote Generation** | Intel TDX Quote Generation Service socket support |
| **Status** | ✅ GPU detected and functional (verified with nvidia-smi) |

```bash
# TDX-specific options
-object '{"qom-type":"tdx-guest","id":"tdx",...}'
-machine q35,kernel_irqchip=split,confidential-guest-support=tdx,memory-backend=mem0
-bios /usr/share/edk2/ovmf/OVMF.inteltdx.fd
```

#### Non-TDX VM (`launch-notdx-gpuAB-vm.sh`)

| Feature | Configuration |
|---------|---|
| **Firmware** | Standard OVMF firmware (CODE + VARS via pflash) |
| **Memory Backend** | Standard RAM backend (`memory-backend-ram`) |
| **Machine Type** | `q35` without confidential settings |
| **Encryption** | None (standard VM) |
| **Quote Generation** | Not supported |
| **Status** | ⚠️ GPU passed through but driver fails to initialize |

```bash
# Standard VM configuration
-drive if=pflash,format=raw,readonly=on,file=/usr/share/edk2/ovmf/OVMF_CODE.fd
-drive if=pflash,format=raw,file=/usr/share/edk2/ovmf/OVMF_VARS.fd
```

### GPU Passthrough Configuration

Both VMs use identical GPU passthrough setup:

#### IOMMU & VFIO Configuration
```bash
-object iommufd,id=iommufd0                             # IOMMU Fd for device I/O management
-device pcie-root-port,id=proto_portB,...               # PCIe root port
-device vfio-pci,host=0001:b0:00.0,iommufd=iommufd0,... # Pass GPU at 0001:b0:00.0
```

#### GPU Details
- **Host PCI Address**: `0001:b0:00.0`
- **Device**: NVIDIA RTX PRO 6000 Blackwell Server Edition (GB202GL)
- **Driver**: vfio-pci (on host), NVIDIA proprietary driver (in guest)
- **Memory**: 97.8 GB VRAM (reported in guest)

A second GPU at `0001:ae:00.0` was not appearing to the host OS due to a hardware issue.

### Console & Monitoring

#### Monitor Socket
```bash
-monitor unix:/tmp/gpuA-monitor.sock,server,nowait
```
Allows interaction with QEMU monitor for VM control (power down, status checks, etc.).

**Note**: Guest VM name is `td-guest` internally and `gpuAM` in scripts.

#### Serial Console Log
```bash
-serial file:/var/log/gpuA-console.log
```
VM console output redirected to log file. Useful for debugging boot issues and driver initialization.

#### Daemonization
```bash
-daemonize
```
VM runs in background after launch.

### Network Configuration

```bash
-netdev bridge,id=nic0_td,br=virbr0
-device virtio-net-pci,netdev=nic0_td,mac=52:54:00:12:34:56
```

- **Bridge**: libvirt default bridge (virbr0)
- **MAC Address**: 52:54:00:12:34:56
- **Driver**: Virtio network device (paravirtualized for performance)

### Storage Configuration

#### Root Filesystem
```bash
-drive file=/mnt/bigdisk/libvirt/images/ubuntu-gpuA.img,if=none,id=virtio-disk0
-device virtio-blk-pci,drive=virtio-disk0
```
- **Image**: Ubuntu guest filesystem
- **Location**: /mnt/bigdisk (high-capacity storage)
- **Driver**: Virtio block (paravirtualized)

#### Cloud-init Seed
```bash
-drive file=/mnt/bigdisk/libvirt/images/td-guest-seed.img,if=none,id=cloud-init,format=raw
-device virtio-blk-pci,drive=cloud-init
```
- **Image**: ISO 9660 cloud-init configuration
- **Purpose**: Guest initialization (hostname, networking, packages, kernel parameters)
- **Format**: Raw ISO image

### Usage

#### Launch TDX VM with GPU
```bash
sudo /opt/tdxexpt/admin-scripts/launch-gpuAB-vm.sh
```

#### Launch Non-TDX VM with GPU
```bash
sudo /opt/tdxexpt/admin-scripts/launch-notdx-gpuAB-vm.sh
```

#### Shutdown VM
```bash
sudo /opt/tdxexpt/admin-scripts/shutdown-gpuAB-vm.sh
```

#### Access VM Console
```bash
# View serial console logs
tail -f /var/log/gpuA-console.log

# Interactive QEMU monitor
socat - UNIX-CONNECT:/tmp/gpuA-monitor.sock
```

#### Verify GPU in Guest
```bash
# Inside guest VM
nvidia-smi          # List GPU devices
lspci | grep NVIDIA # Verify PCIe detection
dmesg | grep nvidia # Check driver initialization
```

### CPU Specification

```bash
-cpu host
```
Passthrough host CPU features directly to guest. Requires:
- Same CPU generation on host
- Nested virtualization support (for TDX)
- TDX CPU support (for TDX VMs)

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
