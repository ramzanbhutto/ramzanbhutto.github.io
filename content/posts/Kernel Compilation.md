+++
title = 'Compile and Build Linux Kernel'
date = '2025-02-05T00:37:50+05:00'
tags = ["Linux","Kernel","Arch"]
+++

## Building Linux Kernel

**The process of building a Linux Kernel can be performed in several steps, but it may take time as per your hardware capabilities.**

---

### Step 1: Download the Source Code 

Visit the [Official Kernel Website](https://www.kernel.org/) and download the latest kernel source code. The downloaded files contains a compressed source code that you need to extract it.

---

### Step 2: Extract the Source Code

After downloading, it is necessary to extract the source code. So, we will achieve this using **tar** command.
~~~ 
 tar xvf linux-6.13.tar.xz
~~~
If you don't have **tar**, then download it using command:

~~~ 
 sudo pacman -S tar  
~~~
 Note: It is recommended to download the latest kernel source code and write it's version correctly while using **tar** command.

---

### Step 3: Install the Required Packages

Make sure you have additional packages to start compilation. To achieve this, you need to install the following **packages**:

```python
 sudo pacman -S git fakeroot ncurses xz bc flex bison base-devel kmod cpio perl binutils util-linux jfsutils e2fsprogs xfsprogs squashfs-tools quota-tools
``` 
---

### Step 4: Configure your kernel

1. Navigate to **linux-6.13** folder:

~~~ 
 cd linux-6.13
~~~
2. Configue your kernel. It is recommended to use your current system's configuration as a base. So write the following commands in order:

  - Use the following commad if you have zcat:

```python
zcat /proc/config.gz > .config
```   
  - Else you have to use following commands to simply copying **config** file:
   ~~~ 
    cp /proc/config.gz ./
    gunzip ./config.gz
    mv config .config
   ~~~
3. Use the following commands to open a menu-driven interface to customize kernel options: 
   ~~~
    make menuconfig
    make xconfig
    `make oldconfig`
   ~~~
4. Make some changes in `.config` file: 
   
   - Open it using command(you can use vim, kate, nano or any other text editor):
   ~~~
   sudo vim .config
   ~~~
   - Make following changes: 
    
     > search for `CONFIG_EXT4_FS=m` and set it to   `CONFIG_EXT4_FS=y`.

---

### Step 5: Start compilation

1. Check for available processing CPU Cores using command: 

~~~
 nproc
~~~
*Note then `n` number of cores shown on the screen*

2. Initiate the compilation process:

~~~
 make -jn   
~~~
**Replace *n* with number of cores that you found using `nproc` command**


***And now your Kernel starts compiling***

If any issue persists during or after running **make -jn**, then make a back up of **.config** file and run the command:

~~~
 make mrproper 
~~~
***make mrproper** resets your entire broken tree back to the initial state.*

---

### Step 6: Install Modules

Insalling Kernel Modules are mandatory and they enhances the kernel's capabilities, hardware support and other features while maintaining system stability and efficiency. You can install kernel modules using commad:

```python
sudo make modules_install
```

---

### Step 7: Install Kernel

**The following are two different methods to install Kernel. Use the first method. If it doesn't work, then use the second method carefully:**

1. Directly install through one command:

~~~
 sudo make install
~~~
2. Use three commands properly and carefully: 
 
  - Copy the Kernel Image:
```python
 sudo cp arch/x86/boot/bzImage  /boot/vmlinuz-linux-custom
```   
  - Copy the System.map file:
```python
 sudo cp System.map  /boot/System.map-linux-custom
```  
  - Copy the configuration file:
```python
 sudo cp .config  /boot/config-linux-custom
```  
---

### Step 8: Update the Bootloader

Depending on your bootloader, you will need to add an entry for the new kernel. I am adding entry for **GRUB** as I uses it.

1. Run the following command to know the UUID of your root partition (it will be in ext4 file system and have the mounting point as /) and copy it:

~~~
 lsblk -f
~~~
2. Open the file using command:
```python
 sudo nvim /etc/grub.d/40_custom
```

3. Add the following content to the above mentioned file at the end:
```python
  menuentry 'Custom Linux Kernel' {  
linux    /boot/vmlinuz-linux-custom  
root=UUID=paste-your-root-partition-uuid-here  
initrd /boot/initramfs-linux.img  
}      
```
---

### Step 9: Generate Initramfs 

*Generating Initramfs is necessary as you have compiled a new custom kernel, installed kernel modules, updated kernel configuration and updated bootloader. It is crucial for boot system. Use the command and make sure to input the correct version as I mentioned **6.13**:*
```python
sudo mkinitcpio -k 6.13-custom -c /etc/mkinitcpio.conf -g /boot/initramfs-linux-custom.img
```
---

### Step 10: Update GRUB configuration

**Use the following command to update it as it will detect our new custom kernel:**
```python
 sudo grub-mkconfig -o /boot/grub/grub.cfg
```
---

**That's it. Configuration, you successfully build, compiled and installed your custom Kernel. Enjoy**

---


