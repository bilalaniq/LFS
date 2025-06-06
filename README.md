# LFS

before LFS you could also click [here](./Recompilation.md) to learn about recompiling linux which will help you understand LFS

## **What Is Linux From Scratch (LFS)?**

**Linux From Scratch** is a project that **teaches you how to build your own custom Linux system from the ground up**, using only **source code** and a minimal host Linux system (like Kali, Ubuntu, etc.).

It’s not a distribution — it’s a **book and guide** that helps you build your own.

---

## **What You Actually Do in LFS**

- You **compile the Linux kernel**
- You **build essential system tools** like `bash`, `gcc`, `make`, `coreutils`
- You **create your own root filesystem**
- You **build a bootable Linux system manually**
- Everything is done **step-by-step using source code**

---

## 📦 Optional Projects After LFS

- **BLFS (Beyond LFS):** Adds things like GUI (X11), networking, browsers, editors, etc.
- **CLFS:** Cross-compile LFS for other architectures (e.g., ARM)
- **ALFS:** Automate the process of building LFS

so this will be an short guide towards LFS lets start

# What You Need

- A working Linux host system (like Kali, Ubuntu, Arch, etc.)
- About 10–20 GB of space
- Around 4–8 hours for a full build (depending on speed and experience)
- CPU have at least four cores and that the system have at least 8 GB of memory. Older systems that do not meet these requirements will still work, but the time to build packages will be significantly longer than documented.
- Bash-3.2 (/bin/sh should be a symbolic or hard link to bash)
- Binutils-2.13.1 (Versions greater than 2.44 are not recommended as they have not been tested)
- Bison-2.7 (/usr/bin/yacc should be a link to bison or a small script that executes bison)
- Coreutils-8.1
- Diffutils-2.8.1
- Findutils-4.2.31
- Gawk-4.0.1 (/usr/bin/awk should be a link to gawk)
- GCC-5.2 including the C++ compiler, g++ (Versions greater than 14.2.0 are not recommended as they have not been tested). C and C++ standard libraries (with headers) must also be present so the C++ compiler can build hosted programs
- Grep-2.5.1a
- Gzip-1.3.12
- Linux Kernel-5.4
- M4-1.4.10
- Make-4.0
- Patch-2.5.4
- Perl-5.8.8
- Python-3.4
- Sed-4.1.5
- Tar-1.22
- Texinfo-5.0
- Xz-5.0.0

here the script to cheak if your system can compile LFS

```bash
cat > version-check.sh << "EOF"
#!/bin/bash
# A script to list version numbers of critical development tools

# If you have tools installed in other directories, adjust PATH here AND
# in ~lfs/.bashrc (section 4.4) as well.

LC_ALL=C
PATH=/usr/bin:/bin

bail() { echo "FATAL: $1"; exit 1; }
grep --version > /dev/null 2> /dev/null || bail "grep does not work"
sed '' /dev/null || bail "sed does not work"
sort   /dev/null || bail "sort does not work"

ver_check()
{
   if ! type -p $2 &>/dev/null
   then
     echo "ERROR: Cannot find $2 ($1)"; return 1;
   fi
   v=$($2 --version 2>&1 | grep -E -o '[0-9]+\.[0-9\.]+[a-z]*' | head -n1)
   if printf '%s\n' $3 $v | sort --version-sort --check &>/dev/null
   then
     printf "OK:    %-9s %-6s >= $3\n" "$1" "$v"; return 0;
   else
     printf "ERROR: %-9s is TOO OLD ($3 or later required)\n" "$1";
     return 1;
   fi
}

ver_kernel()
{
   kver=$(uname -r | grep -E -o '^[0-9\.]+')
   if printf '%s\n' $1 $kver | sort --version-sort --check &>/dev/null
   then
     printf "OK:    Linux Kernel $kver >= $1\n"; return 0;
   else
     printf "ERROR: Linux Kernel ($kver) is TOO OLD ($1 or later required)\n" "$kver";
     return 1;
   fi
}

# Coreutils first because --version-sort needs Coreutils >= 7.0
ver_check Coreutils      sort     8.1 || bail "Coreutils too old, stop"
ver_check Bash           bash     3.2
ver_check Binutils       ld       2.13.1
ver_check Bison          bison    2.7
ver_check Diffutils      diff     2.8.1
ver_check Findutils      find     4.2.31
ver_check Gawk           gawk     4.0.1
ver_check GCC            gcc      5.2
ver_check "GCC (C++)"    g++      5.2
ver_check Grep           grep     2.5.1a
ver_check Gzip           gzip     1.3.12
ver_check M4             m4       1.4.10
ver_check Make           make     4.0
ver_check Patch          patch    2.5.4
ver_check Perl           perl     5.8.8
ver_check Python         python3  3.4
ver_check Sed            sed      4.1.5
ver_check Tar            tar      1.22
ver_check Texinfo        texi2any 5.0
ver_check Xz             xz       5.0.0
ver_kernel 5.4

if mount | grep -q 'devpts on /dev/pts' && [ -e /dev/ptmx ]
then echo "OK:    Linux Kernel supports UNIX 98 PTY";
else echo "ERROR: Linux Kernel does NOT support UNIX 98 PTY"; fi

alias_check() {
   if $1 --version 2>&1 | grep -qi $2
   then printf "OK:    %-4s is $2\n" "$1";
   else printf "ERROR: %-4s is NOT $2\n" "$1"; fi
}
echo "Aliases:"
alias_check awk GNU
alias_check yacc Bison
alias_check sh Bash

echo "Compiler check:"
if printf "int main(){}" | g++ -x c++ -
then echo "OK:    g++ works";
else echo "ERROR: g++ does NOT work"; fi
rm -f a.out

if [ "$(nproc)" = "" ]; then
   echo "ERROR: nproc is not available or it produces empty output"
else
   echo "OK: nproc reports $(nproc) logical cores are available"
fi
EOF

bash version-check.sh
```

if you get an result like this then you are good to go and if you encounter any error then get along an fix that error

```bash
bash version-check.sh
OK:    Coreutils 9.5    >= 8.1
OK:    Bash      5.2.32 >= 3.2
OK:    Binutils  2.43.1 >= 2.13.1
OK:    Bison     3.8.2  >= 2.7

OK:    Diffutils 3.10   >= 2.8.1
OK:    Findutils 4.10.0 >= 4.2.31
OK:    Gawk      5.2.1  >= 4.0.1
OK:    GCC       14.2.0 >= 5.2
OK:    GCC (C++) 14.2.0 >= 5.2
OK:    Grep      3.11   >= 2.5.1a
OK:    Gzip      1.12   >= 1.3.12
OK:    M4        1.4.19 >= 1.4.10
OK:    Make      4.3    >= 4.0
OK:    Patch     2.7.6  >= 2.5.4
OK:    Perl      5.40.1 >= 5.8.8
OK:    Python    3.13.2 >= 3.4
OK:    Sed       4.9    >= 4.1.5
OK:    Tar       1.35   >= 1.22
OK:    Texinfo   7.1.1  >= 5.0
OK:    Xz        5.6.4  >= 5.0.0
OK:    Linux Kernel 6.11.2 >= 5.4
OK:    Linux Kernel supports UNIX 98 PTY
Aliases:
OK:    awk  is GNU
OK:    yacc is Bison
OK:    sh   is Bash
Compiler check:
OK:    g++ works
OK: nproc reports 2 logical cores are available
```

now if all is good now lets cover LSF step by step

---

<br>

# Creating a New Partition

Sure! Here’s a summarized version of the information, tailored to your LFS setup in a **VM** running **Kali Linux**:

### **Partitioning Overview for LFS**:

1. **Partitioning Basics**:

   - LFS (Linux From Scratch) needs a **dedicated partition** for installation.
   - A **minimum of 10 GB** is recommended for the LFS system. If you plan to install additional software later, a **30 GB** partition is a good size for growth.
   - You’ll also need space for temporary storage during the build process.

2. **Swap Space**:

   - Swap space is helpful for managing memory during compilation. A **small swap partition (1–2 GB)** is useful if your VM has limited RAM.
   - Swap space is optional if you have enough RAM. However, if you intend to use **hibernation**, the swap partition should be as large as the system's RAM.

3. **Creating Partitions**:

   - Inside your VM, use partitioning tools like `fdisk` or `cfdisk` to create the necessary partitions.
   - You can create:

     - **Root Partition** (`/`): Around 10-30 GB, depending on your needs.
     - **Swap Partition** (if needed): Around 1-2 GB.
     - Optionally, **additional partitions** like `/boot` (200 MB) and `/home` for shared data across systems.

4. **File Systems and Formatting**:

   - After creating the partitions, format them using tools like `mkfs.ext4` for the root partition and `mkswap` for swap.

5. **Mounting Partitions**:

   - Mount the root partition to `/mnt/lfs` to begin the LFS build process:

     ```bash
     sudo mount /dev/sda1 /mnt/lfs  # Adjust partition name as needed
     ```

   - Enable swap using `swapon`.

6. **LFS System Design**:

   - For **experienced users**, more complex partitioning schemes are possible, like using **RAID** or **LVM**, but these are not recommended for beginners.
   - The LFS root partition should be designed to hold the LFS build files and the base system.

7. **Other Partition Considerations**:

   - **/boot**: Recommended for storing bootloader and kernel files (around 200 MB).
   - **/home**: Optional, useful for storing user data across builds.
   - **/opt**: For additional packages like KDE or Texlive (5-10 GB).
   - **/usr/src**: Useful for storing BLFS (Beyond LFS) source files and builds (30-50 GB recommended).

8. **Important Notes**:

   - You’ll need to add entries for the partitions in the **/etc/fstab** file, so they are mounted automatically at boot time.
   - **The GRUB BIOS Partition** is needed for booting systems with a **GUID Partition Table (GPT)**. This is typically 1 MB and doesn’t need to be formatted.

---

### **VM**:

- Since i am using a **VM** for your LFS build, you will **allocate disk space** for your virtual machine. This will simulate the partitioning process just as if you were doing it on a physical machine.
- After partitioning inside the VM, follow the steps for formatting, mounting, and building the LFS system.

---

you don't **need** to use **GRUB with UEFI** for your LFS setup if you're working inside a **VM**. Here's why:

### 1. **UEFI and GRUB:**

- **UEFI (Unified Extensible Firmware Interface)** is a modern replacement for BIOS and is commonly used in systems to initialize hardware and load operating systems.
- **GRUB (Grand Unified Bootloader)** is typically used as the bootloader, which loads the operating system once the system firmware (BIOS/UEFI) has initialized the hardware.

For a typical LFS installation:

- If you're working with **legacy BIOS** mode in your VM, you do **not** need UEFI, and GRUB will simply load the kernel directly.
- If you're using **UEFI** in your VM (which is common for newer systems), you **may need** to set up a separate **EFI system partition** and install GRUB for UEFI booting.

### 2. **For VM Setup:**

- **Legacy BIOS Mode**: If your VM is set to use **BIOS**, you can install GRUB without worrying about UEFI. GRUB will be used to boot the LFS system from the root partition.
- **UEFI Mode**: If your VM is set to use **UEFI**, you’ll need to create an **EFI system partition** (usually around 100-200 MB), install GRUB for UEFI, and ensure the VM is booting in UEFI mode.

### **Steps for VM Setup (Based on Boot Mode):**

#### **1. If Using Legacy BIOS (No UEFI):**

- No need to create an EFI partition.
- Install GRUB directly on the root partition (e.g., `/dev/sda`).
- GRUB will work as a bootloader and will load the LFS system without requiring UEFI.

#### **2. If Using UEFI:**

- You **need to create an EFI partition** (100-200 MB) during partitioning.
- GRUB needs to be installed to the EFI partition.
- Configure the system to boot from UEFI and set up GRUB to work with UEFI, making sure the UEFI firmware on your VM is properly configured for this.

### **Conclusion:**

- **For most LFS setups in a VM**, **Legacy BIOS** mode is sufficient, and you don't need to worry about UEFI or the EFI partition.
- However, if you want to work with **UEFI** (for modern hardware compatibility), you will need to set up an EFI partition and install GRUB for UEFI booting.

---

<br>
<br>

> ![NOTE]
> i will be using kalilinux on VM to compile an linux(LFS)

<br>

Identify the disk you want to use for LFS (e.g., /dev/sda).

```bash
sudo fdisk -l
```

---

```bash
┌──(kali㉿kali)-[~]
└─$ sudo fdisk -l
Disk /dev/sda: 80.09 GiB, 86000000000 bytes, 167968750 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x3d57c577

Device     Boot Start       End   Sectors  Size Id Type
/dev/sda1  *     2048 167968749 167966702 80.1G 83 Linux
```

I have one large partition (/dev/sda1) taking up the entire 80GB disk.

Launch fdisk to modify the partition table:

```bash
sudo fdisk /dev/sda
```

```bash
──(kali㉿kali)-[~]
└─$ sudo fdisk /dev/sda

[sudo] password for kali:

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.


Command (m for help):

```

now there is an big issue "This disk is currently in use - repartitioning is probably a bad idea.

You are trying to modify the main disk (/dev/sda) while Kali is running from it. Since the Kali system is installed on /dev/sda1, it's actively being used by the OS right now. You cannot safely resize or repartition a disk that is currently mounted and in use — doing so may corrupt the file system or crash the system.

fdisk won't let you shrink or safely modify it while it's in use.

You need to boot into a Live environment (like Kali Live or GParted Live ISO), which doesn't mount /dev/sda1, so you can safely resize it.
