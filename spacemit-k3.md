---
layout: default
title: SpacemiT K3
nav_order: 3
permalink: /spacemit-k3/
---
<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- Copyright (C) 2026 Thalles Campagnani -->

# SpacemiT K3 — Firmware & Blob Map

Overview of all hardware blocks, firmware images, open-source boot and real-time components, and signature/integrity requirements on the **SpacemiT K3** RISC-V SoC (8× X100 + 8× A100 cores; reference board: **SpacemiT K3 Pico ITX**).

Every closed-source **on-SoC** firmware blob on the K3 (DDR PMU training, GPU, and VPU) now has an independent clean-room specification linked below.

---

## On-SoC Hardware & Firmware Status

| Hardware Block | Firmware / Target Interface | Status | Cryptographic Signature Enforced? | Clean-Room / Open-Source Link |
|---|---|---|---|---|
| **DDR Training** (LPDDR5 & LPDDR4X, dual controller) | ARCv2 EM PMU full-training & quickboot firmware (embedded in FSBL) | **Clean-room spec available** | **No** on reference boards (FSBL has RSA-2048 structures, `CONFIG_SPACEMIT_SECURE_BOARD` disabled; vendor keys provided) | [SpacemiT_K3_DDR](https://github.com/NoBlobs-docsClean/SpacemiT_K3_DDR) |
| **GPU — PowerVR Rogue BXM-4-64 MC1** (BVNC `36.56.104.183`) | `powervr/rogue_36.56.104.183_v1.fw` (mainline `drm/imagination` ABI 1.1) | **Clean-room spec available** | **No** (`RGX_FW_SIGNED` disabled; BVNC/header check only) | [Imagination_BXM-4-64](https://github.com/NoBlobs-docsClean/Imagination_BXM-4-64) |
| **VPU — Arm China Linlon V5** (`0x56635002`, 4 cores) | `linlon-v52_v76-80-2/*.fwb` (MVE 3.5 protocol, GPL-2.0 `mvx` driver, RASC ISA) | **Clean-room spec available** | **No** (180-byte `mvx_fw_header` + size check only) | [Linlon_V5](https://github.com/NoBlobs-docsClean/Linlon_V5) |
| **RCPU / ESOS** (2× RT24 / CVA6 RV64GC, 614.4 MHz) | `esos.itb` (`rt24_os0_rcpu.elf` / `rt24_os1_rcpu.elf`, started by SPL; Linux attaches) | **Open source** (Apache-2.0) | **No** (CRC32 FIT only; `CONFIG_SPL_FIT_SIGNATURE` disabled) | [spacemit-com/esos](https://github.com/spacemit-com/esos) / [esos-lite](https://github.com/spacemit-com/esos-lite) |
| **Boot Chain** (FSBL/SPL, OpenSBI, U-Boot or EDK2 UEFI, optional OP-TEE) | `FSBL.bin`, `fw_dynamic.itb`, `u-boot.itb` or EDK2 FIT | **Open source** | **No** on reference boards (RSA-2048 `ROTPK`/`cert0`/`signature1` present but unenforced; vendor keys provided in U-Boot tree) | [Mainline U-Boot](https://github.com/u-boot/u-boot), [Mainline OpenSBI](https://github.com/riscv-software-src/opensbi) ([Vendor U-Boot](https://github.com/spacemit-com/uboot-2022.10), [Vendor OpenSBI](https://github.com/spacemit-com/opensbi), [Vendor EDK2](https://github.com/spacemit-com/edk2)) |
| **AI (60 TOPS)** | 8× A100 CPU cores with SpacemiT-IME2 instructions (no dedicated NPU) | **No firmware blob** | N/A | — |
| **ISP/CCIC, V2D, JPU/JPEG, DPU/DSI/DP/eDP, UFS, PCIe, USB3, GMAC (4×), PMIC, RTC, eSPI, CAN-FD** | Hardware + kernel drivers | **No firmware blob** | N/A | — |

### Off-SoC Board Peripherals (External Chips on K3 Pico ITX)

| Board Module | Interface | Firmware Status | Cryptographic Signature Enforced? |
|---|---|---|---|
| **Embedded Controller (ChromeOS EC)** | eSPI (`google,cros-ec-espi`) | Closed, **external board-specific MCU** (outside the K3 SoC) | **No** (updater checks manifest MD5/SHA-256 only) |
| **Wi-Fi — Realtek RTL8852BE** | PCIe (`rtw89_8852be`) | Closed, **external chip outside the K3 SoC** | **Yes** (`rtw89` hardware secure boot by eFuse) |
| **Bluetooth — Realtek RTL8852BU** | USB (`0bda:b85b`, `btrtl`) | Closed, **external chip outside the K3 SoC** | **No** (EPATCH header and config magic check only) |


---

## Open-Source RCPU / ESOS Details

The K3 integrates **two real-time RT24 cores** (CVA6, RV64GC, 614.4 MHz, 5 MB RAM each) running independent instances of **ESOS** (*Energy Service OS*, based on RT-Thread). Because ESOS is open source under Apache-2.0 (and vendor packages even ship unstripped debug builds), it does not require clean-room reverse engineering:

- **Source repositories:** [spacemit-com/esos](https://github.com/spacemit-com/esos) (`bsp/spacemit/platform/rt24`) and [spacemit-com/esos-lite](https://github.com/spacemit-com/esos-lite) (Apache-2.0)
- **Packaging & startup (`[P][O]`):** Packaged as a dual-core FIT image (`esos.itb`, containing `rt24_os0_rcpu.elf` and `rt24_os1_rcpu.elf` plus per-board DTBs). Unlike the K1, **U-Boot SPL** loads and starts both RT24 cores during early boot; Linux does not load the image and instead attaches at runtime (`drivers/remoteproc/k3-rproc.c`, `rcpu_rproc1`).
- **System interfaces (`[P][O]`):** Communicates with Linux via `virtio`/`rpmsg` (e.g., `adma-service`) and provides platform clock, regulator, RTC, powerkey, and reset services via **RPMI over OpenSBI MPXY** (`sbi-mpxy-mbox`).

---

## Key Public References

- [spacemit-com/docs-chip](https://github.com/spacemit-com/docs-chip) — Public SpacemiT K3 Datasheet and User Manual
- [spacemit-com/uboot-2022.10](https://github.com/spacemit-com/uboot-2022.10) — Public U-Boot SPL & K3 board configs (`k3-br-v1.0.y`)
- [spacemit-com/opensbi](https://github.com/spacemit-com/opensbi) — OpenSBI with RPMI over SBI-MPXY (`k3-br-v1.0.y`)
- [spacemit-com/edk2](https://github.com/spacemit-com/edk2) & [edk2-platforms](https://github.com/spacemit-com/edk2-platforms) — UEFI firmware (`Platform/Spacemit/K3`)
- [spacemit-com/esos](https://github.com/spacemit-com/esos) & [spacemit-com/esos-lite](https://github.com/spacemit-com/esos-lite) — Open-source dual-RT24 RCPU firmware (Apache-2.0)
- [ARM-software/rasc-devkit](https://github.com/ARM-software/rasc-devkit) — Public GPLv2 toolchain for the VPU's RASC CPUs
