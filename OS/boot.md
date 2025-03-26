# Boot Sequence Overview

The boot sequence is the process a computer follows to load the operating system (OS) when powered on.

## Boot Process Steps

1. **Power On**
2. **CPU loads BIOS/UEFI**
   - BIOS (Basic Input/Output System)
   - UEFI (Unified Extensible Firmware Interface)
3. **Power-On Self-Test (POST)**
   - The BIOS/UEFI firmware runs hardware checks (CPU, RAM, storage, etc.)
   - If errors occur, beep codes or on-screen messages appear
4. **BIOS/UEFI Loads Bootloader from MBR or EFI partition**
4. **Bootloader Execution**
   - The bootloader (e.g., GRUB for Linux, Windows Boot Manager for Windows) loads the OS kernel
   - In dual-boot systems, the bootloader lets you choose an OS
5. **Kernel Initialization**
   - The OS kernel takes over, initializes drivers, and starts essential services
   - Example: Linux loads initramfs (temporary root filesystem) if needed

## BIOS vs UEFI Comparison

| Feature              | BIOS (Legacy)               | UEFI (Modern)               |
|----------------------|----------------------------|----------------------------|
| Boot Mode            | MBR (≤2TB)                 | GPT (>2TB)                 |
| Partitions           | Max 4 primary              | Up to 128                  |
| Interface            | 16-bit (text-based)        | 32/64-bit (GUI possible)   |
| Boot Speed           | Slower (POST checks)       | Faster (parallel init)     |
| Secure Boot          | No                         | Yes (blocks malware)       |
| Network Support      | No                         | Yes (PXE boot)             |
| Drivers              | Limited                    | Extensible (modular)       |
| Compatibility        | Older OSs (Win 7 & earlier)| Newer OSs (Win 8+, Linux)  |

## UEFI with CSM (Legacy BIOS Support)

- Most modern motherboards with UEFI firmware include a CSM (Compatibility Support Module) option in their settings
- When CSM is enabled, the system can boot in Legacy BIOS mode for older operating systems (e.g., Windows 7, XP, or some Linux distros)

## MBR (Master Boot Record) Overview

MBR is a legacy partitioning scheme used to organize disk storage. It has been largely replaced by GPT in modern systems.

### GPT (GUID Partition Table) vs. MBR Comparison

| Feature         | GPT                          | MBR                  |
|-----------------|------------------------------|----------------------|
| Max Disk Size   | 9.4 ZB (theoretical)         | 2 TB                |
| Max Partitions  | 128                          | 4 (primary)         |
| Boot Mode       | UEFI Only                    | BIOS/Legacy         |
| Recovery        | Redundant partition table    | Single point of failure |
| OS Support      | Windows 8+/Linux (64-bit)    | Windows 7 and older |

## GPT Partition Structure

When you initialize a disk as GPT, Windows typically creates these partitions:

| Partition       | Purpose                          | Size           | Required? |
|-----------------|----------------------------------|----------------|-----------|
| EFI System Partition (ESP) | Stores bootloaders (UEFI) | 100–500 MB     | ✅ Yes (for UEFI boot) |
| Microsoft Reserved (MSR) | System operations (GPT management) | 16 MB | ✅ Yes (Windows only) |
| Windows (C:)    | Main OS installation             | Remaining space | ✅ Yes    |
| Recovery Partition | Reset/repair tools            | 500 MB–1 GB    | ❌ Optional |

## Microsoft Reserved Partition (MSR)

The MSR is a small, hidden partition created by Windows during installation on GPT disks when using UEFI boot mode.

### Purpose of the MSR Partition
- Supports GPT Disk Management
- Required for dynamic disks (a legacy Windows feature)
- Helps with partition alignment and future Windows updates
- Reserved for System Use (e.g., BitLocker metadata, recovery tools)
- Only for GPT Disks (not present on MBR disks)

> **Note:** Deleting the MSR partition is not recommended as Windows may fail to update or boot. If missing, Windows Setup will recreate it automatically.

## EFI System Partition (ESP) Requirements

- The UEFI standard mandates that the ESP must use FAT32 (or rarely FAT16)
- UEFI firmware has built-in FAT32 drivers for reading boot files
- Other filesystems (NTFS, exFAT, ext4) wouldn't be recognized by UEFI

> **Technical Note:** GPT/MBR are partitioning schemes (manage partitions), while FAT32/NTFS/exFAT are file systems (manage files on partitions).

## How a Computer Detects MBR vs GPT

1. **Check the First Sector (Boot Process)**
   - BIOS/UEFI reads the first sector (512 bytes)
   - If last 2 bytes are `0x55AA` (MBR signature): treated as MBR disk
   - If no `0x55AA` or GPT signature exists: checks for GPT header at LBA 1

2. **GPT Protective MBR (Hybrid Check)**
   - GPT disks include a protective MBR to prevent legacy tool corruption
   - Contains dummy partition entry and `0x55AA` signature
   - UEFI ignores protective MBR and looks for real GPT header at LBA 1

3. **Firmware Behavior**
   | Firmware        | Checks For               | Boot Method               |
   |-----------------|--------------------------|---------------------------|
   | Legacy BIOS     | Only checks for MBR      | Boots via MBR bootloader  |
   | UEFI Firmware   | Checks for GPT header    | Boots via EFI bootloader  |

### GPT vs. MBR Signature Comparison

| Feature        | GPT Signature                     | MBR Signature       |
|----------------|-----------------------------------|---------------------|
| Location       | LBA 1 (Primary), Last Sector (Backup) | First sector (LBA 0) |
| Magic Value    | "EFI PART" (45 46 49 20 50 41 52 54) | `0x55AA` (last 2 bytes) |
| Purpose        | Confirms GPT disk structure       | Confirms MBR bootability |
| Backup Exists? | Yes (at end of disk)              | No                   |

## Bootloader Configuration

- Boot priority can be configured in BIOS/UEFI settings (e.g., USB → SSD → HDD)
- In dual-boot systems, the boot sequence determines which EFI partition to boot from


## How to Check Your System
### Windows:
1. Open Disk Management → If your boot disk is GPT, you're using UEFI
   - If it's MBR, you're likely in Legacy BIOS mode
2. Or, run:
   ```cmd
   msinfo32
   ```
   - Check "BIOS Mode" (UEFI or Legacy).


### Linux:
```console
ls /sys/firmware/efi
```
If the folder exists → UEFI. If not → Legacy BIOS
