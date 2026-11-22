---
layout: default
title: Home
nav_order: 1
permalink: /
---
<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- Copyright (C) 2026 Thalles Campagnani -->

# NoBlobs Clean-Room Documentation Portal

Independent hardware, register, ISA, and ABI specifications enabling **clean-room open-source firmware** for modern RISC-V SoCs — starting with the **SpacemiT K1** and **SpacemiT K3** platforms.

---

## Why This Organization Exists

Modern SoCs ship with small embedded microcontrollers inside subsystems such as DDR memory controllers, GPUs, and video codecs. Even when the Linux kernel driver is open source, these subsystems often depend on proprietary binary firmware blobs with no public hardware manual.

**NoBlobs-docsClean** serves as the **public specification and audit firewall** between hardware analysis ([**NoBlobs-dirtyTeam**](https://github.com/NoBlobs-dirtyTeam), private repositories) and independent open-source firmware implementation ([**NoBlobs**](https://github.com/NoBlobs)). Every document here is filtered and audited so that implementers can write open-source firmware from scratch without ever accessing proprietary binaries, disassemblies, or vendor closed-source code.

Every specification in this organization covers **100% of the closed-source on-SoC firmware blobs** on the **SpacemiT K1** and **SpacemiT K3** (DDR training, GPU, and VPU) — while the remaining on-SoC processors (such as the real-time RCPU running ESOS) and boot stages are already open source.

---

## SoC Specification and Repositories

### [SpacemiT K1](spacemit-k1.md)

Full SoC firmware and blob map covering LPDDR4X training, PowerVR BXE-2-32 GPU, Arm China Linlon V5 VPU (2 cores), open-source RCPU/ESOS, blob-free peripherals, signature enforcement, and off-SoC board modules.

Repositories:

| Hardware IP | Target Host ABI / Driver | Repository |
|---|---|---|
| **SpacemiT K1 DDR** (LPDDR4 / LPDDR4X) | Mainline U-Boot SPL (`ddr_fw.bin` slot) | [NoBlobs-docsClean/SpacemiT_K1_DDR](https://github.com/NoBlobs-docsClean/SpacemiT_K1_DDR) |
| **PowerVR Rogue BXE-2-32** (`36.29.52.182`) | Mainline Linux `powervr` DRM + Mesa `pvr` | [NoBlobs-docsClean/Imagination_BXE-2-32](https://github.com/NoBlobs-docsClean/Imagination_BXE-2-32) |
| **Arm China Linlon V5 VPU** (`0x56635002`, 2 cores) | Vendor Linux `mvx` driver | [NoBlobs-docsClean/Linlon_V5](https://github.com/NoBlobs-docsClean/Linlon_V5) |

### [SpacemiT K3](spacemit-k3.md)

Full SoC firmware and blob map covering LPDDR5 PMU training, PowerVR BXM-4-64 GPU, Arm China Linlon V5 VPU (4 cores), open-source dual-RT24 RCPU/ESOS, open-source boot chain (OpenSBI + U-Boot / EDK2 UEFI), signature enforcement, and off-SoC modules.

Repositories:

| Hardware IP | Target Host ABI / Driver | Repository |
|---|---|---|
| **SpacemiT K3 DDR** (LPDDR5 PMU ARC) | Vendor U-Boot SPL | [NoBlobs-docsClean/SpacemiT_K3_DDR](https://github.com/NoBlobs-docsClean/SpacemiT_K3_DDR) |
| **PowerVR Rogue BXM-4-64** (`36.56.104.183`) | Mainline Linux `powervr` DRM + Mesa `pvr` | [NoBlobs-docsClean/Imagination_BXM-4-64](https://github.com/NoBlobs-docsClean/Imagination_BXM-4-64) |
| **Arm China Linlon V5 VPU** (`0x56635002`, 4 cores) | Vendor Linux `mvx` driver | [NoBlobs-docsClean/Linlon_V5](https://github.com/NoBlobs-docsClean/Linlon_V5) |

---

## Clean-Room Guarantee & Provenance

All documents published under **NoBlobs-docsClean** follow a strict clean-room firewall (see [Clean-Room Process](clean-room.md)):

- **No proprietary code or binaries:** Zero copied code, disassembly listings, decompiled symbols, internal firmware strings, or private internal layouts.
- **Fact-only specifications:** Hardware register maps, bus protocols, public host driver ABIs, memory layouts, public ISAs, and empirically verified silicon behavior.
- **Explicit provenance tags:**
  - **`[P]` Public** — Derived from open-source Linux/U-Boot headers, public ISAs, Device Trees, and datasheets.
  - **`[O]` Observed** — Verified empirically on real K1 / K3 hardware boards.
  - **`[D]` Derived** — Hardware register programming and behavioral requirements needed to satisfy `[P]` and `[O]` on real silicon.

---

## License

All documentation in this organization is licensed under **Creative Commons Attribution-ShareAlike 4.0 International (`CC-BY-SA-4.0`)** unless noted otherwise in individual repository notices (referenced public kernel ABI definitions and public toolchains retain their original open-source licenses).
