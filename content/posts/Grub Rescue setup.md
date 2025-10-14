+++
title = 'Grub Rescue Setup'
date = '2025-02-05T00:37:50+05:00'
tags = ["Linux","GRUB","boot"]
+++
# Understanding GRUB Rescue and How to Fix It  
*(A Guide to Diagnosing and Resolving Boot Errors)*  

---

## Introduction  
GRUB (Grand Unified Bootloader) is a critical component of Linux systems, responsible for loading the operating system. However, encountering the **GRUB rescue prompt** can be alarming. This blog explains why this happens and provides two practical methods to resolve it.  

---

## Why Does GRUB Rescue Appear?  
The GRUB rescue prompt appears when the bootloader fails to locate critical files or configurations. Common causes include:  
1. **Corrupted or Broken GRUB Installation**: Occurs after interrupted updates or failed OS installations.  
2. **Incorrect Partition Changes**: Modifying disk partitions (e.g., deleting/resizing) can break GRUB’s path.  
3. **Missing OS or Boot Files**: Accidental deletion of `/boot` or misconfigured kernel updates.  
4. **Disk Errors**: Hardware failures or filesystem corruption.  

---

## Method 1: On-the-Spot Fix Using GRUB Rescue Commands  

If you have access to the GRUB rescue prompt, try these steps:  


### Step 1: Identify Partitions  
List available partitions using:  
~~~
    ls
~~~

### Step 2: Locate the Boot Partition
Check each partition for the /boot/grub directory:
```python
ls (hdX,msdosY)/boot/grub
```
>Continue this process until you locate the /boot/grub directory. Then, replace X (disk number) and Y (partition number) based on your system.


### Step 3: Set the Correct Root and Prefix
Set the correct root and prefix:
```python
set root=(hdX,msdosY)  
```
```python
set prefix=(hdX,msdosY)/boot/grub
```

### Step 4: Load the Normal Modules
~~~
insmod normal
~~~
~~~
normal
~~~
>This should bring up the standard GRUB menu, allowing you to boot into your operating system.


### Step 5: Reinstall GRUB
Once booted, it's crucial to reinstall GRUB to prevent future issues:
```python
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
> It generated grub configuration file for arch-based distros, for debian based distros, you can use the following command: 
~~~
sudo grub-update
~~~
Now reinstall the GRUB Bootloader to the specified disk:
```python
sudo grub-install /dev/sdaX 
```
>It is recommended to regenerate the grub configuration after this step.


---

## Method 2: GRUB Setup using a Live USB

This method involves booting into a Live USB environment, mounting the necessary partitions, and reinstalling GRUB manually. Follow these steps carefully:  


### **Step 1: Boot into Live USB**  
1. Create a Live USB using tools like [Rufus](https://rufus.ie/) or [BalenaEtcher](https://www.balena.io/etcher/).  
2. Boot into the Live environment by selecting the USB drive in your BIOS/UEFI menu.  


### **Step 2: Identify Partitions**  
Use lsblk to list all disks and partitions:    
~~~
lsblk
~~~

### **Step3: Mount the root partition**
Mounts the root partition (/dev/nvme0n1p4) to the /mnt directory. Replace /dev/nvme0n1p4 with your actual root partition identifier.
```python
sudo mount  /dev/nvme0n1p4 /mnt
```

### **Step 4: Mount the boot partition**
Mounts the boot partition (/dev/nvme0n1p3) to /mnt/boot. Adjust /dev/nvme0n1p3 to match your boot partition.
```python
sudo mount  /dev/nvme0n1p3 /mnt/boot
```

### **Step 5: Bind Mount System Directories**
Bind /dev, /proc, /sys, and EFI variables to the chroot environment:

#### **i:  /dev directory**
Bind the /dev directory to /mnt/dev for device access in the chroot environment.
```python
sudo mount  --bind  /dev  /mnt/dev
```
#### **ii:  /proc directory**
Bind the /proc directory to /mnt/proc for process information.
```python
sudo mount --bind /proc /mnt/proc
```
#### **iii:  /sys directory**
Bind the /sys directory to /mnt/sys for system information.
```python
sudo mount  --bind  /sys  /mnt/sys
```
#### **iv:  efi variables**
Binds EFI variables to /mnt/sys/firmware/efi/efivars for UEFI systems.
```python
sudo mount  --bind  /sys/firmware/efi/efivars  /mnt/sys/firmware/efi/efivars
```
#### **v:  /dev/pts (optional for nvme or gpt file systems)**
If your system uses the pts filesystem or you want a pseudo-terminal support.
```python
sudo mount --bind /dev/pts /mnt/dev/pts
```


### **Step 6: Chroot into the System**
Switches the root directory to /mnt, allowing you to work on the installed system. Enter the chroot environment:
```python
sudo chroot /mnt
```
### **Step 7: Reinstall GRUB**
(Arch Based) Install the GRUB bootloader for UEFI systems, specifying the target architecture, EFI directory, and a bootloader identifier.

```python
sudo grub-install  - -target=x86_64-efi  - -efi-directory=/boot/efi  - -bootloader-id=GRUB
```

### **Step 8: Generate GRUB Configuration file**
Generate a new GRUB configuration file, detecting available operating systems and kernels.
```python
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### **Step 9: Exit chroot Environment**
Exits the chroot environment, returning to the Live USB session.
~~~
exit
~~~

### **Step 9: Unmount Filesystems**
Recursively unmounts all filesystems mounted under /mnt.
```python
sudo umount -R /mnt
```

### **Step 10: Reboot the System**
Restarts the system. Remove the Live USB to boot into your repaired system.
~~~
reboot
~~~

### **Step 11: Regenerate GRUB Configuration File**
After booting into your system, it's advisable to regenerate the GRUB configuration file to ensure all operating systems are detected:
```python
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### **Step 12: Reboot**
~~~
sudo reboot now
~~~
---

> Facing the "grub rescue" prompt can be resolved using methods like `insmod` or a `Live USB`. Utilize the Arch Linux community, forums, and the Arch Wiki for support. Proactively addressing boot issues and leveraging resources ensures a stable system. Seek help or consult documentation for further assistance.

