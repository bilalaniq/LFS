# How to Recompile the Linux Kernel

## Overview

In this tutorial, let’s discuss the kernel and learn how to recompile it. Often, users decide to customize or build their kernel for different reasons. Frequently, we just want to change our kernel configuration to suit our preferences or those of our systems.

Before we begin compiling, we must ensure that the following prerequisites are met to successfully install a new kernel.

Firstly, we must check our current kernel version:

```bash
┌──(kali㉿kali)-[~]
└─$ uname -r
6.11.2-amd64
```

---

```bash
┌──(kali㉿kali)-[~/linux_kernel/linux-6.7]
└─$ uname -a
Linux kali 6.11.2-amd64 #1 SMP PREEMPT_DYNAMIC Kali 6.11.2-1kali1 (2024-10-15) x86_64 GNU/Linux

```

Let’s install the following required dependencies:

```bash
sudo apt-get install build-essential git fakeroot ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison dwarves
```

We need these dependencies for the following reasons:

<br>

| **Dependency**    | **Function**                                                                                                           |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `build-essential` | Meta-package including GCC, G++, make, and other essential tools needed for compiling software.                        |
| `git`             | Version control system used to track changes in the source code and manage development history.                        |
| `fakeroot`        | Allows users to simulate root privileges for file manipulation during packaging without actual root access.            |
| `ncurses-dev`     | Provides libraries for building text-based user interfaces in a terminal.                                              |
| `xz-utils`        | Offers compression and decompression tools for `.xz` files, commonly used for kernel source archives.                  |
| `libssl-dev`      | Contains headers and libraries for building programs that use SSL/TLS via OpenSSL.                                     |
| `bc`              | Arbitrary precision calculator language, often used in scripts for math operations.                                    |
| `flex`            | A lexical analyzer generator used to produce scanners (tokenizers) for compilers and other tools.                      |
| `libelf-dev`      | Provides APIs to handle ELF (Executable and Linkable Format) files, used by the kernel and other binary-related tools. |
| `bison`           | Parser generator that converts grammar descriptions into C code, often used alongside `flex`.                          |
| `dwarves`         | Tools to extract and process debugging information (such as `pahole`), useful during kernel and low-level development. |

<br>
<br>

---

We must ensure that when we’re recompiling the Kernel, we should have at least 15 GB of free space on our hard drive so that the kernel compilation may take place seamlessly.

you can see this by using this command

```bash
┌──(kali㉿kali)-[~]
└─$ df -l
Filesystem     1K-blocks     Used Available Use% Mounted on
udev             1944264        0   1944264   0% /dev
tmpfs             401648     1248    400400   1% /run
/dev/sda1       82083148 17895080  59972520  23% /
tmpfs            2008224        4   2008220   1% /dev/shm
tmpfs               5120        0      5120   0% /run/lock
tmpfs               1024        0      1024   0% /run/credentials/systemd-journald.service
tmpfs            2008224        0   2008224   0% /tmp
tmpfs             401644      156    401488   1% /run/user/1000

```

## Kernel Source

1. **Stable Kernels**:

   - These are the **most reliable** and **tested** versions of the kernel.
   - They are regularly updated with bug fixes, new features, and improvements.
   - Suitable for most users who want a stable system.

2. **Mainline Kernels**:

   - These are the **latest and cutting-edge** versions of the kernel.
   - They have the newest features but might not be as **stable** because they are still in active development.
   - Ideal if you want the newest features but don't mind some risks with stability.

3. **Hardened Kernels**:

   - These kernels are **extra secure**.
   - They have additional patches and settings to protect your system from potential security threats.
   - Great for users who need extra security.

4. **Long-Term Support (LTS) Kernels**:

   - These versions are **supported for a long time**, often for several years.
   - They focus on **stability** and **security** rather than new features.
   - Perfect for enterprise or production environments where reliability is key.

5. **Real-Time Kernels**:

   - These kernels are designed for tasks that require **precise timing** and **predictable behavior**.
   - They are used in **critical systems** like robotics or industrial automation where low latency is needed.

6. **Zen Kernels**:

   - These kernels are optimized for **performance** on desktops and workstations.
   - They focus on making your system faster and more responsive, especially for things like **gaming** or **multimedia tasks**.

---

In simple terms, choose:

- **Stable kernels** for everyday use.
- **Mainline kernels** for the latest features.
- **Hardened kernels** for better security.
- **LTS kernels** for long-term stability.
- **Real-time kernels** for precision tasks.
- **Zen kernels** for high performance in desktop environments.

---

<br>
<br>

we’ll use the mainline version, which we can access through:

```bash
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.7.tar.xz
```

---

## Configuring the Kernel

Now that we’ve downloaded the kernel version we need, let’s extract it:

```bash
tar xvf linux-6.7.tar.xz
```

```bash
cd linux-6.7
```

```bash
cp -v /boot/config-$(uname -r) .config
```

copies your system’s current kernel configuration into your working directory as `.config`.

That config file includes **a lot of extra drivers and modules**, many of which you **don’t need**.

To speed up kernel compilation, you can use:

```bash
make localmodconfig
```

This command checks which kernel modules are **currently loaded** on your system and updates `.config` to include **only those**.

This approach is **faster**, but it’s best to **only use the resulting kernel on the same machine** it was built on.

after running this command you will get this risponse

```bash
..
...
....
#
# configuration written to .config
#
```

Then, we must make these modifications to the configuration:

```bash
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable SYSTEM_REVOCATION_KEYS
scripts/config --set-str CONFIG_SYSTEM_TRUSTED_KEYS ""
```

- **`SYSTEM_TRUSTED_KEYS`**: Used to verify module signatures and secure boot signatures; they ensure only trusted code runs in the kernel.
- **`SYSTEM_REVOCATION_KEYS`**: Used to identify and block previously trusted certificates that are now revoked due to compromise or policy change.
- **`CONFIG_SYSTEM_TRUSTED_KEYS`**: Specifies the file or string of X.509 certificates to be compiled into the kernel as trusted keys for verification.
  Would you like to keep these features or disable them for a leaner, custom kernel build?

now commands

- `scripts/config --disable SYSTEM_TRUSTED_KEYS`: Disables the use of trusted certificate keys in the kernel.
- `scripts/config --disable SYSTEM_REVOCATION_KEYS`: Disables the use of certificate revocation keys in the kernel.
- `scripts/config --set-str CONFIG_SYSTEM_TRUSTED_KEYS ""`: Sets the trusted keys string to empty, re

- fakeroot makeving any built-in trusted keys.

---

<br>
<br>

# Installing the Kernel

After, completing kernel configuration, let’s start compiling it.

Here’s a simple explanation of each step:

**`fakeroot make`** – Pretends to be root so the build process can create files (like kernel modules) with root-like permissions **without needing real root access**.

This helps the kernel build process, which needs special permissions to create certain files.

If you run just make (without fakeroot), it might fail silently

```bash
fakeroot make
```

---

To check if the build was successful, run echo $?:

- If it shows 0, everything is OK.
- If it's not 0, something went wrong

```bash
echo $?
0
```

---

We can expedite the process by maximizing the number of cores utilized. To find out the available cores we run:

```
nproc
8
```

**`nproc`** – Tells you how many **CPU cores** your system has (to speed up compilation).

---

**`fakeroot make -j3`** – Compiles the kernel using **3 CPU cores** to make it faster.

```bash
fakeroot make -j3
```

now the compilation will occur using three cores

you will get an risponse like

```bash
Kernel: arch/x86/boot/bzImage is ready  (#1)
```

there is no error

```bash
┌──(kali㉿kali)-[~/linux_kernel/linux-6.7]
└─$ echo $?
0

```

---

Next, let’s install the kernel modules:

**`sudo make modules_install`** – Installs the **kernel modules** (pieces of code like drivers).

```bash
sudo make modules_install
```

---

**`sudo make install`** – Installs the **kernel itself**, so it can be booted later.

```bash
sudo make install
```

If the installation is successful, we should see “done” displayed on the terminal:

```bash
┌──(kali㉿kali)-[~/linux_kernel/linux-6.7]
└─$ sudo make install
  INSTALL /boot
run-parts: executing /etc/kernel/postinst.d/initramfs-tools 6.7.0 /boot/vmlinuz-6.7.0
update-initramfs: Generating /boot/initrd.img-6.7.0
run-parts: executing /etc/kernel/postinst.d/zz-update-grub 6.7.0 /boot/vmlinuz-6.7.0
Generating grub configuration file ...
Found theme: /boot/grub/themes/kali/theme.txt
Found background image: /usr/share/images/desktop-base/desktop-grub.png
Found linux image: /boot/vmlinuz-6.11.2-amd64
Found initrd image: /boot/initrd.img-6.11.2-amd64
Found linux image: /boot/vmlinuz-6.7.0
Found initrd image: /boot/initrd.img-6.7.0
Found linux image: /boot/vmlinuz-6.7.0.old
Found initrd image: /boot/initrd.img-6.7.0
Warning: os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
done
```

rebbot the system

```bash
sudo reboot
```

After rebooting, we can check our kernel version again if it has updated to the one we’ve just installed:

```bash
┌──(kali㉿kali)-[~]
└─$ uname -r
6.11.2-amd64
```

it is same no changes why this is because system is still booting into the old kernel. we have to tell the system to to boot into the new kernel

to see all the compiled linux kernels

```bash
┌──(kali㉿kali)-[~]
└─$ ls /boot/vmlinuz-*

/boot/vmlinuz-6.11.2-amd64  /boot/vmlinuz-6.7.0  /boot/vmlinuz-6.7.0.old

```

To make sure that GRUB recognizes the new kernel, run

```bash
sudo update-grub
```

and then reboot the system

but if for some reason does not select the right kenel. like in my case it is selecting the old compiled version of linux-6.7.0

so we will do it maually

Open the GRUB configuration file:

```bash
sudo nano /etc/default/grub
```

Look for the line starting with `GRUB_DEFAULT`.

```bash
GRUB_DEFAULT=0
```

You should modify it to point to the kernel you want to boot by default. For example, if you want to boot into 6.7.0, you can set it too

change it depending upon your linux version

or you could select the linux when booting

## conclusion:

we learned what the **kernel** is and what it does. We also saw why someone might want to **recompile** the kernel. Before doing that, it’s important to make sure you have **enough storage space** and the **correct packages** installed.

Finally, how long it takes to compile the kernel depends mostly on how **powerful your computer** is.

---

<br>
<br>

now i will compile the inux kenel to just print `Welcome Bilal` so you could validate that linux kernel has been recompiled

first we will print msg in the kernel looging for that you wold have to edit the `/init/main.c` file

you could use vim or nano i will br using mousepad

```bash
mousepad init/main.c
```

you need to find the `void start_kernel(void)` function and write this in the start of the function

```c
printk(KERN_EMERG "Welcome Bilal - booting kernel...\n");
```

---

Customizing the Linux kernel with a graphical flavor is possible, but note that the kernel itself does not support rich GUIs—it's responsible for hardware, processes, memory, etc. However, you can make customizations at the kernel level, boot level, and desktop level to give it a visual, personalized experience

---


now if you have learned about the recompilation lets do an assignment click [here](./assignment.md) 
