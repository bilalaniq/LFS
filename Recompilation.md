# How to Recompile the Linux Kernel

## Overview

In this tutorial, let’s discuss the kernel and learn how to recompile it. Often, users decide to customize or build their kernel for different reasons. Frequently, we just want to change our kernel configuration to suit our preferences or those of our systems.

Before we begin compiling, we must ensure that the following prerequisites are met to successfully install a new kernel.

Firstly, we must check our current kernel version:

```bash
ubuntu@ubuntu-virtual-machine:~$ uname -r
5.19.0-38-generic
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

