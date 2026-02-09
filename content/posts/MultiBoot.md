---
title: "Multi-Boot Setup: Ubuntu + Arch Linux + Windows 11"
date: "2026-02-09"
draft: false
showToc: true
Author: "Muhammad Ramzan"
tags: ["linux", "ubuntu", "arch-linux", "windows", "multi-boot", "grub"]
description: "A comprehensive guide to setting up a multi-boot system with Ubuntu, Arch Linux, and Windows 11 on the same machine."
---

# Multi-Boot Setup: Ubuntu + Arch Linux + Windows 11

---

## Introduction

Multi-booting allows you to run multiple operating systems on a single computer, giving you the flexibility to switch between different environments. This guide will walk you through setting up Ubuntu on a laptop that already has Arch Linux and Windows 11 installed.

### Why Multi-Boot?

- **Flexibility**: Access different OS features and tools
- **Development**: Test software across multiple platforms
- **Learning**: Explore different Linux distributions
- **Compatibility**: Use Windows-only software when needed
- **Backup**: Have alternative systems if one fails

---

## Prerequisites

### Before You Start

- **Backup your data**: Always backup important files before partitioning
- **Ubuntu ISO**: Download from [ubuntu.com](https://ubuntu.com/download)
- **Bootable USB**: At least 4GB USB drive
- **Free disk space**: Minimum 25GB (50GB+ recommended)
- **UEFI system**: Modern systems use UEFI instead of legacy BIOS
- **Secure Boot**: May need to disable for Linux installations
- **Internet connection**: For updates and troubleshooting

### Tools Needed

- **Rufus** (Windows) or **Etcher** (Linux) for creating bootable USB
- **GParted** or built-in partition manager
- **Booted USB** of Ubuntu for installation

---

## Understanding Your Current Setup

### Check Existing Partitions

From your existing Arch Linux or Windows system, check current partition layout:

```bash
# From Arch Linux
sudo fdisk -l
# or
lsblk -f

# From Windows (PowerShell as Administrator)
Get-Partition
# or use Disk Management GUI
```

### Typical Multi-Boot Layout

```sh
Device             Start        End   Sectors   Size Type
/dev/nvme0n1p1      2048  514091007 514088960 245.1G Microsoft basic data
/dev/nvme0n1p2 514091008  515586047   1495040   730M Windows recovery environment
/dev/nvme0n1p3 515588096  517431295   1843200   900M EFI System
/dev/nvme0n1p4 517431296  811032575 293601280   140G Linux filesystem
/dev/nvme0n1p5 811032576  833857535  22824960  10.9G Linux swap
/dev/nvme0n1p6 833857536  833890303     32768    16M Microsoft reserved
/dev/nvme0n1p7 833890304 1000214527 166324224  79.3G Microsoft basic data
```
---

## Step 1: Prepare Disk Space

### Option A: Shrink Existing Partition (Recommended)

#### From Windows:

1. Press `Win + X` → Select "Disk Management"
2. Right-click on a partition with free space (usually Windows C:)
3. Select "Shrink Volume"
4. Enter amount to shrink (at least 50GB = 51200 MB)
5. Click "Shrink"

#### From Arch Linux:

```bash
# Install GParted if not available
sudo pacman -S gparted

# Launch GParted
sudo gparted

# Select partition → Resize/Move → Shrink from end
# Leave space unallocated for Ubuntu
```
---

## Step 2: Create Ubuntu Bootable USB

### From Windows:

1. Download **Rufus** from [rufus.ie](https://rufus.ie)
2. Insert USB drive
3. Open Rufus:
   - **Device**: Select your USB drive
   - **Boot selection**: Select Ubuntu ISO
   - **Partition scheme**: GPT
   - **Target system**: UEFI
4. Click "START"
5. Select "Write in ISO Image mode" (recommended)

### From Arch Linux:

```bash
# Using dd (replace X with your USB device letter)
sudo dd bs=4M if=/path/to/ubuntu.iso of=/dev/nvmeX status=progress oflag=sync
```

**Warning**: Double-check the device name! Using the wrong device will erase data.

---

## Step 3: BIOS/UEFI Configuration

### Access BIOS/UEFI

Restart your computer and press the appropriate key during startup:
- **Common menu keys**: F2, F10, F12, DEL, ESC
- **Specific brands**:
  - Dell: F2 or F12
  - HP: F10 or ESC
  - Lenovo: F1 or F2
  - ASUS: F2 or DEL
  - Acer: F2 or DEL

### Recommended Settings

```
Secure Boot: Disabled (or add Ubuntu keys)
Boot Mode: UEFI (not Legacy/CSM)
Fast Boot: Disabled
Boot Order: USB as first priority (temporarily)
```

---

## Step 4: Boot from USB

1. Insert the Ubuntu USB drive
2. Restart your computer
3. Press the boot menu key
4. Select your USB drive from the boot menu
5. Choose "Try or Install Ubuntu"

---

## Step 5: Install Ubuntu

### Installation Steps

1. **Select Language**: Choose your preferred language

2. **Keyboard Layout**: Select your keyboard layout

3. **Updates and Software**:
   - Select "Normal installation"
   - Check "Download updates while installing Ubuntu"
   - Check "Install third-party software" (for WiFi, graphics, etc.)

4. **Installation Type**: **CRITICAL STEP**
   - Select "Something else" (manual partitioning)
   - **DO NOT** select "Erase disk and install Ubuntu"

### Partition Layout for Ubuntu

Create the following partitions in the free space:

#### Recommended Partition Scheme:

```bash
# Root partition
Mount point: /
Type: ext4
Size: 30-50GB
Flag: -

# Home partition (optional but recommended)
Mount point: /home
Type: ext4
Size: Remaining space (or fixed size)
Flag: -

# Swap partition (if not already present)
Mount point: [none - swap area]
Type: swap
Size: 2-8GB (equal to RAM for hibernation)
Flag: swap
```

#### Minimal Partition Scheme:

```bash
# Single root partition (simpler)
Mount point: /
Type: ext4
Size: All available space
Flag: -
```

### Detailed Steps:

1. **Verify Partitions**: Double-check before proceeding

2. **Click "Install Now"**

3. **Set Timezone**: Select your location

4. **Create User Account**:
   - Your name
   - Computer name
   - Username
   - Password
   - Select login options

5. **Wait for Installation**: Usually takes 15-30 minutes

6. **Restart**: Remove USB when prompted

---

## Step 6: Post-Installation Configuration

### Update GRUB to Detect All Operating Systems

After booting into Ubuntu:

```bash
# Update package list
sudo apt update

# Install os-prober (to detect other OS)
sudo apt install os-prober

# Enable os-prober in GRUB
sudo nvim /etc/default/grub

# Add or uncomment this line:
GRUB_DISABLE_OS_PROBER=false

# Save and exit 

# Update GRUB
sudo update-grub

# Reboot
sudo reboot
```

### If Ubuntu Doesn't Appear in Boot Menu

Boot into Arch Linux and update its GRUB:

```bash
# From Arch Linux
sudo os-prober
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo reboot
```

### If Windows Doesn't Appear

```bash
# Install ntfs-3g for NTFS support
sudo apt install ntfs-3g

# Remount EFI partition (if needed)
sudo mount /dev/X /boot/efi

# Update GRUB again
sudo update-grub
```

---

## Step 7: Boot Menu Management

### Change Default Boot OS

```bash
# Edit GRUB configuration
sudo nano /etc/default/grub

# Change default entry (0 = first, 1 = second, etc.)
GRUB_DEFAULT=0

# Or set by name
GRUB_DEFAULT="Ubuntu"

# Change timeout (seconds)
GRUB_TIMEOUT=10

# Update GRUB
sudo update-grub
```

### Install GRUB Customizer (GUI)

```bash
sudo add-apt-repository ppa:danielrichter2007/grub-customizer
sudo apt update
sudo apt install grub-customizer
```

---

## Summary

Multi-booting Ubuntu alongside Arch Linux and Windows 11 gives you:
- **Flexibility** to use different operating systems
- **Learning opportunities** across multiple platforms
- **Compatibility** with various software ecosystems
- **Backup options** if one system fails

---
