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

This was done by injecting a marker string into the VMs memory by e.g. declaring a variable in the python interpreter
```python
a = 'SECRET_MARKER_STRING'
```
On the host, we then connect to the unix monitor socket by `socat - UNIX_CONNECT:/path/to/monitor.sock` and dump the memory of the VM into a file by issuing the command `dump-guest-memory <dump_filename>`. This results in a binary file we can search through for the marker string e.g. `grep -ao SECRET_MARKER_STRING <dump_filename>`.

## GPU Attestation

The Nvidia Attestation SDK is an open source project for implementing and validating TEEs on Nvidia hardware. It provides C and Rust APIs and a CLI (`nvattest`) for generating and verifying attestation claims from a Nvidia GPU.

## Example Workloads

### Single GPU benchmarks

#### Notes
- The non-confidential VM fails to connect to the GPU at the moment so I haven't been able to add results from it.
- The second GPU in the guest VMs is still offline due to hardware issues

#### Conclusions
- Compute in the CVM is not any slower than the bare metal host. Once the data is in the GPU memory everything works as usual
- Data copy from host to device is 10x slower in the CVM. Need to confirm TDX is the only factor affecting it, but sounds plausible.
- The variance in the sciml benchmarks has been estimated from 3 repeated runs of the benchmark.
- The two SciML benchmarks move roughly the same amount of data (7.8 GB vs 8.4 GB), but `dms_structure` has a higher number of data copies (8093 vs 226) which could be the reason it performs worse on the CVM.

Benchmark | Metric | Host (bare metal, CC off) | Guest (CC off) | TDX guest (CC on) | TDX Guest / host
-- | -- | -- | -- | -- | --
[STREAM (NVIDIA)](https://docs.nvidia.com/nvidia-hpc-benchmarks/index.html) | Copy (GB/s) | 1,477.8 | | 1,477.5 | 1.00
[STREAM (NVIDIA)](https://docs.nvidia.com/nvidia-hpc-benchmarks/index.html)  | Scale (GB/s) | 1,477.4 | | 1,477.0 | 1.00
[STREAM (NVIDIA)](https://docs.nvidia.com/nvidia-hpc-benchmarks/index.html)  | Add (GB/s) | 1,500.6 | | 1,501.4 | 1.00
[STREAM (NVIDIA)](https://docs.nvidia.com/nvidia-hpc-benchmarks/index.html) | Triad (GB/s) | 1,501.9 | | 1,501.1 | 1.00
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | H→D memcpy, CE (GB/s) | 54.6 | | 5.15 | 0.094 (~10.6× slower)
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | D→H memcpy, CE (GB/s) | 56.5 | | 5.64 | 0.100 (~10.0× slower)
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | H↔D bidirectional, CE (GB/s) | 48.6 | | waived (CC) | —
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | Host–device latency, SM (ns) | 1,067 | | waived (CC) | —
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | GPU↔GPU P2P, CE (GB/s) | 38.0 | | n/a (1 GPU) | —
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | Device-local copy, CE (GB/s) | 730.4 | | 728.8 | 1.00
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | Device-local copy, SM (GB/s) | 720.3 | | 720.4 | 1.00
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | Device-local copy, TMA (GB/s) | 733.2 | | 732.3 | 1.00
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | Device-local read-only, SM (GB/s) | 1,539.5 | | 1,540.4 | 1.00
[nvbandwidth](https://github.com/NVIDIA/nvbandwidth) | Device-local write-only, SM (GB/s) | 1,489.3 | | 1,457.4 | 0.98
[sciml-bench dms_structure](https://github.com/stfc-sciml/sciml-bench/blob/master/sciml_bench/docs/markdown/benchmarks/dms_structure.md) | Time (s), mean ± sd, n=3 | 22.08 ± 0.08 | | 30.01 ± 0.52 | 1.36 (+35.9%)
[sciml-bench dms_structure](https://github.com/stfc-sciml/sciml-bench/blob/master/sciml_bench/docs/markdown/benchmarks/dms_structure.md) | Accuracy (%), mean ± sd, n=3 | 79.5 ± 0.5 | | 78.8 ± 0.2 | within run-to-run variation
[sciml-bench em_denoise](https://github.com/stfc-sciml/sciml-bench/blob/master/sciml_bench/docs/markdown/benchmarks/em_denoise.md) | Time (s), mean ± sd, n=3 | 22.07 ± 0.27 | | 23.05 ± 0.21 | 1.044 (+4.4%)
[sciml-bench em_denoise](https://github.com/stfc-sciml/sciml-bench/blob/master/sciml_bench/docs/markdown/benchmarks/em_denoise.md) | MSE, mean ± sd, n=3 | 0.017 ± 0.0002 | | 0.017 ± 0.0002 | 1.00
