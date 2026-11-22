---
layout: default
title: SpacemiT K1
nav_order: 2
permalink: /spacemit-k1/
---
<!-- SPDX-License-Identifier: CC-BY-SA-4.0 -->
<!-- Copyright (C) 2026 Thalles Campagnani -->

# SpacemiT K1 — Firmware & Blob Map

Overview of all hardware blocks, firmware images, open-source components, and signature/integrity requirements on the **SpacemiT Key Stone K1** RISC-V SoC (4× X60-AI + 4× X60 cores; reference boards: **Banana Pi BPI-F3** 8 GB and 16 GB).

Every closed-source **on-SoC** firmware blob on the K1 (DDR training, GPU, and VPU) now has an independent clean-room specification linked below.

---

## On-SoC Hardware & Firmware Status

| Hardware Block | Firmware / Target Interface | Status | Cryptographic Signature Enforced? | Clean-Room / Open-Source Link |
|---|---|---|---|---|
| **DDR Training** (LPDDR4 / LPDDR4X) | `ddr_fw.bin` (loaded by U-Boot SPL into SRAM at `0xc082d000`) | **Clean-room spec available** | **Yes** — inside FSBL (RSA-2048/SHA-256; signing keys provided by vendor) | [SpacemiT_K1_DDR](https://noblobs-docsclean.github.io/SpacemiT_K1_DDR/) |
| **GPU — PowerVR Rogue BXE-2-32** (BVNC `36.29.52.182`) | `powervr/rogue_36.29.52.182_v1.fw` (mainline `powervr` DRM + Mesa `pvr`) | **Clean-room spec available** | **No** (`RGX_FW_SIGNED` disabled; BVNC/header check only) | [Imagination_BXE-2-32](https://noblobs-docsclean.github.io/Imagination_BXE-2-32/) |
| **VPU — Arm China Linlon V5** (`0x56635002`, 2 cores) | `linlon-v52_v76-80-2/*.fwb` (MVE 3.5 protocol, GPL-2.0 `mvx` driver, RASC ISA) | **Clean-room spec available** | **No** (180-byte `mvx_fw_header` + size check only) | [Linlon_V5](https://noblobs-docsclean.github.io/Linlon_V5/) |
| **RCPU / ESOS** (Nuclei N308 RV32, 245 MHz) | `esos.elf` (RT-Thread + OpenAMP/rpmsg via Linux `remoteproc`) | **Open source** (Apache-2.0) | **No** (standard ELF loaded by `k1x-rproc.c`) | [spacemit-com/esos](https://github.com/spacemit-com/esos) |
| **Boot Chain** (FSBL/SPL, OpenSBI, U-Boot or EDK2 UEFI) | `FSBL.bin`, `fw_dynamic.itb`, `u-boot.itb` or EDK2 FIT | **Open source** | **Yes** — FSBL RSA-2048/SHA-256 (`cert0` + `signature1`; vendor keys provided in U-Boot tree) | [Mainline U-Boot](https://github.com/u-boot/u-boot), [Mainline OpenSBI](https://github.com/riscv-software-src/opensbi), ([Vendor U-Boot](https://github.com/spacemit-com/uboot-2022.10), [Vendor OpenSBI](https://github.com/spacemit-com/opensbi), [Vendor EDK2](https://github.com/spacemit-com/edk2)) |
| **AI ("Daoyi" 2 TOPS)** | 4× X60-AI CPU cores with SpacemiT-IME instructions (no dedicated NPU) | **No firmware blob** | N/A | — |
| **ISP / CPP** (camera) | Kernel driver + userspace pipeline (no binary firmware loaded) | **No firmware blob** | N/A | — |
| **JPU, V2D, DPU/HDMI, PMIC (SPM8821), USB, PCIe, Ethernet** | Hardware + kernel drivers | **No firmware blob** | N/A | — |

### Off-SoC Board Peripherals (External Chips)

| Board Module (Banana Pi BPI-F3) | Interface | Firmware Status | Cryptographic Signature Enforced? |
|---|---|---|---|
| **Wi-Fi — Realtek RTL8852BS** | SDIO (`sdhci1`) | Closed, **external chip outside the K1 SoC** | **Yes** (hardware secure boot / chip eFuses; rejects modified images) |
| **Bluetooth — Realtek RTL8852BS** | UART2 (`/dev/ttyS2`, H5 3-wire, 115200 → 1.5 Mbps) | Closed, **external chip outside the K1 SoC** | **No** (EPATCH format header check only) |

*(Other K1 boards may populate alternative external radio modules.)*

---

## Open-Source RCPU / ESOS Details

The K1 integrates a real-time **Nuclei N308** (RV32IMAFC, 245 MHz, 256 KB SRAM + shared DDR) running **ESOS** (*Energy Service OS*, based on RT-Thread + OpenAMP/rpmsg). Because ESOS is open source under Apache-2.0, it does not require clean-room reverse engineering:

- **Source repository:** [spacemit-com/esos](https://github.com/spacemit-com/esos) (`bsp/spacemit/platform/n308/k1-x`)
- **Exposed rpmsg services (`[O]`):** `rcpu-pwr-management-service`, `i2c-service`, `rir-service`, `ruart-service0`, `ruart-service1`, `adma-service`
- **Linux loader (`[P]`):** `drivers/remoteproc/spacemit/k1x-rproc.c` (`rcpu_rproc`, `firmware-name = "esos.elf"`)

---

## Key Public References

- [SpacemiT K1 User Manual (PDF)](https://cdn-resource.spacemit.com/file/chip/K1/K1_User_Manual_en.pdf)
- [spacemit-com/esos](https://github.com/spacemit-com/esos) — Open-source RCPU / ESOS firmware (Apache-2.0)
- [spacemit-com/spacemit-firmware](https://github.com/spacemit-com/spacemit-firmware) — Vendor firmware distribution repository
- [ARM-software/rasc-devkit](https://github.com/ARM-software/rasc-devkit) — Public GPLv2 toolchain for the VPU's RASC CPUs
