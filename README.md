# meta-tools-oss

Qualcomm IPQ SoC meta tools for firmware image generation, flash partition management, and single-image packing.

## Supported Platforms

| Platform | Arch | 32/64-bit | Flash Types |
|----------|------|-----------|-------------|
| ipq40xx | ARMv7 | 32-bit | nor, nand, emmc, norplusnand, norplusemmc |
| ipq806x | ARMv7 | 32-bit | nor, nand, emmc, norplusnand, norplusemmc |
| ipq807x | ARMv8 | 32/64-bit | nor, nand, tiny-nor, emmc, norplusnand, norplusemmc, tiny-nor-debug |
| ipq5018 | ARMv7 | 32/64-bit | nor, nand, tiny-nor, emmc, norplusnand, norplusemmc, tiny-nor-debug |
| ipq6018 | ARMv8 | 32/64-bit | nor, nand, tiny-nor, emmc, norplusnand, norplusemmc, tiny-nor-debug |
| ipq9048 | ARMv8 | 32/64-bit | nor, nand, tiny-nor, emmc, norplusnand, norplusemmc, tiny-nor-debug |
| ipq9574 | ARMv8 | 32/64-bit | nor, nand, tiny-nor, emmc, norplusnand, norplusemmc, tiny-nor-debug |
| ipq5332 | ARMv8 | 32/64-bit | nand, nor, tiny-nor, emmc, norplusnand, norplusemmc, tiny-nor-debug |
| ipq5210 | ARMv8 | 32/64-bit | nor, nand, emmc, norplusnand, norplusemmc, norplusnand-gpt, norplusemmc-gpt |
| ipq5424 | ARMv8 | 32/64-bit | nor, nand, emmc, norplusnand, norplusemmc, norplusnand-gpt, norplusemmc-gpt, tiny-nor, tiny-nor-debug |
| ipq9650 | ARMv8 | 32/64-bit | nor, nand, emmc, norplusnand, norplusemmc, norplusnand-gpt, norplusemmc-gpt |

## Requirements

- **Python 3.6+**
- `mkimage` (U-Boot tools)
- `dtc` (Device Tree Compiler, >= 1.4)
- `objcopy` / `ld` (binutils, for MBN conversion)

## Directory Structure

```
meta-tools-oss/
├── ipq40xx/             # Platform configs: config.xml, cdt/, flash_partition/, bootconfig/
├── ipq806x/
├── ipq807x/
├── ipq5018/
├── ipq6018/
├── ipq5332/
├── ipq5424/
├── ipq5210/
├── ipq9048/
├── ipq9574/
├── ipq9650/
├── scripts/             # Shared tool scripts
│   ├── gen_cdt_bin.py           # CDT binary generator
│   ├── gen_flash_partition_bin.py # Partition table generator
│   ├── gen_bootconfig_bin.py    # Bootconfig generator
│   ├── gen_bootldr_bin.py       # Bootloader binary generator
│   ├── gen_xblconfig_bin.py     # XBL config / QC config generator
│   ├── elftombn.py              # ELF to MBN converter
│   ├── gen_its.py               # ITS/ITB image generator
│   ├── create_multielf.py       # Multi-ELF (MELF) creator
│   ├── Gen_xbl_nand_elf.py      # NAND XBL image generator
│   └── ...
├── prepareSingleImage.py  # Main image preparation tool
├── pack.py                # Legacy single-image packer
└── pack_v3.py             # Modern single-image packer (v3)
```

## Usage

### 1. prepareSingleImage.py — Prepare Firmware Images

Generate CDT, partition tables, bootconfig, and convert bootloader images.

```bash
# Generate CDT and partition table for ipq807x eMMC
python3 prepareSingleImage.py --arch ipq807x --fltype emmc --gencdt --genpart

# Generate multiple flash types at once
python3 prepareSingleImage.py --arch ipq6018 --fltype nor,nand,emmc --gencdt --genpart

# Generate bootconfig binary
python3 prepareSingleImage.py --arch ipq40xx --genbootconf

# Convert u-boot.elf to u-boot.mbn (supported platforms only)
python3 prepareSingleImage.py --arch ipq807x --genmbn

# Convert LK bootloader ELF to MBN (ipq807x only)
python3 prepareSingleImage.py --arch ipq807x --genmbn --lk

# Generate merged ELF for TME-L patch (ipq5424/ipq5210/ipq9650)
python3 prepareSingleImage.py --arch ipq9650 --genmelf

# Generate bootloader binary
python3 prepareSingleImage.py --arch ipq5424 --genbootldr

# Use custom input/output directory
python3 prepareSingleImage.py --arch ipq9574 --fltype nor,nand --gencdt --genpart --in ./my_output/
```

#### Platform-Specific Features

| Feature | Platforms |
|---------|-----------|
| `--genmbn` (ELF→MBN) | ipq807x, ipq6018, ipq5018, ipq9574, ipq5332, ipq5424, ipq5210, ipq9650, ipq9048, ipq806x |
| `--lk` (LK MBN) | ipq807x |
| `--genxblcfg` | ipq5424 |
| `--genqccfg` | ipq5210, ipq9650 |
| `--genmelf` (merged ELF) | ipq5424, ipq5210, ipq9650 |
| `--gentfambn` (TF-A MBN) | ipq5210, ipq9650 |
| `--genopteembn` (OP-TEE MBN) | ipq5210, ipq9650 |
| `--genbootconf` | ipq40xx, ipq806x, ipq807x, ipq6018, ipq5018, ipq9574, ipq5332, ipq5424, ipq5210, ipq9650, ipq9048 |
| `--gencdt` | All platforms |
| `--genpart` | All platforms |
| `--genbootldr` | All platforms |
| `--genlicense` | All platforms |

### 2. pack_v3.py — Create Flashable Single Image

Pack multiple firmware images into a single U-Boot flashable image.

```bash
# Basic usage: pack images into a single flash image
python3 pack_v3.py --arch ipq807x --fltype nand \
    --srcPath ./meta-tools-oss \
    --inImage ./my-images \
    --outImage ./output

# Pack for multiple flash types
python3 pack_v3.py --arch ipq6018 --fltype nor,nand,emmc \
    --srcPath ./meta-tools-oss \
    --inImage ./my-images \
    --outImage ./output

# 64-bit mode (append _64 to arch name)
python3 pack_v3.py --arch ipq807x_64 --fltype nand \
    --srcPath ./meta-tools-oss \
    --inImage ./my-images \
    --outImage ./output

# Generate only bootloader images (no full image)
python3 pack_v3.py --arch ipq5424 --fltype emmc \
    --srcPath ./meta-tools-oss \
    --inImage ./my-images \
    --bootldr

# RDP-based split images (ipq5210/ipq9650)
python3 pack_v3.py --arch ipq9650 --fltype emmc \
    --srcPath ./meta-tools-oss \
    --inImage ./my-images \
    --outImage ./output \
    --split_by_rdp

# Use specific flash layout
python3 pack_v3.py --arch ipq9574 --fltype nand \
    --srcPath ./meta-tools-oss \
    --inImage ./my-images \
    --outImage ./output \
    --flayout my_layout
```

#### pack_v3.py Output

The tool generates a `.img` file that can be loaded into SDRAM and flashed from U-Boot:

```
u-boot> imgaddr=0x88000000 source $imgaddr:script
```

### 3. Flash Scripts (in scripts/)

Individual utility scripts for specific tasks:

```bash
# Generate CDT binary from platform config
python3 scripts/gen_cdt_bin.py -c ipq807x/config.xml -o ./output/

# Generate flash partition table binary
python3 scripts/gen_flash_partition_bin.py -c ipq807x/config.xml -f nand -o ./output/

# Generate bootconfig binary
python3 scripts/gen_bootconfig_bin.py -c ipq807x/config.xml -o ./output/

# Generate bootloader binary
python3 scripts/gen_bootldr_bin.py -c ipq5424/config.xml -o ./output/

# Convert ELF to MBN
python3 scripts/elftombn.py -a ipq807x -f u-boot.elf -o u-boot.mbn -v 6
```

## Platform Config (config.xml)

Each platform directory contains a `config.xml` that defines:

- **SOC info**: architecture name
- **Flash parameters**: page size, block size, total blocks per flash type
- **Board mappings**: machid → board name, memory type, SPI NAND support
- **Copy images**: paths for BOOT, TZ, NHSS, RPM, WIFIFW images
- **Bootloader components**: qclib, tfa_bl31, uboot, optee paths (newer platforms)

## u-boot flash notes

For the generated flash script to work on target, U-Boot must have:
- `CONFIG_FIT` — FIT image support
- `CONFIG_SYS_HUSH_PARSER` — bash-style scripting
- `CONFIG_CMD_XIMG` — sub-image extraction
- `CONFIG_IPQ_MIBIB_RELOAD` — MIBIB partition reload (MIBIB-based platforms)
- Flash driver support (`CONFIG_CMD_NAND`, `CONFIG_CMD_SF`, etc.)
