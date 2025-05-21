# Project requirements:

In this project you are required to explore the Linux operating system at the kernel level by selecting a
core operating system concept, identifying it in the kernel source and making a meaningful modification
(method you have discussed in class) to its implementation in the kernel source code.

You will set up a development environment, identify and edit the relevant part of the Linux kernel
source, recompile the kernel, and successfully boot into the modified version to observe and test the
performance effects of the change.

The report will include the following sections:

• The steps to download, compile and boot from the compiled kernel (with screenshots)
• The process of identification of your selected kernel component within the source code
• Selection of an alternative methodology for modifying the chosen kernel component
• Criteria for evaluating the performance of kernel before and after modification, and discussion on
results of comparative performance evaluation

we will modify the following kernel components:

1. CPU scheduling algorithm
2. Page replacement algorithm
3. Page table implementation (hierarchical, hashed or inverted)

---

<br><br>

# CPU scheduling algorithm

if you want to learn about CPU scheduling plz click [here](https://github.com/bilalaniq/notes/tree/main/os)

Here is a list of **CPU scheduling algorithms**:

1. **First-Come, First-Served (FCFS)**
2. **Shortest Job First (SJF)**

   - Non-preemptive
   - Preemptive (also known as Shortest Remaining Time First - SRTF)

3. **Round Robin (RR)**
4. **Priority Scheduling**

   - Preemptive
   - Non-preemptive

5. **Multilevel Queue Scheduling**
6. **Multilevel Feedback Queue Scheduling**
7. **Fair Share Scheduling**
8. **Completely Fair Scheduler (CFS)** _(used in modern Linux)_
9. **Earliest Deadline First (EDF)** _(used in real-time systems)_
10. **Rate-Monotonic Scheduling (RMS)** _(real-time scheduling)_
11. **Lottery Scheduling**
12. **Stride Scheduling**
13. **O(1) Scheduler** _(used in older Linux kernels)_
14. **Weighted Fair Queuing (WFQ)**
15. **Proportional Share Scheduling**

so we will do benchmarking for each and see which is faster and good

Modern Linux uses a modular and extensible Completely Fair Scheduler (CFS) by default. Real-time scheduling classes like `SCHED_FIFO` or `SCHED_RR` are also available, but only used when explicitly set.

Check System-Wide Default Scheduler

Most processes run under the default policy SCHED_OTHER, which maps to CFS.

```bash
ps -e -o pid,comm,policy | grep -v POLICY
```

output:

```bash
┌──(root㉿kali)-[/sys/kernel/debug]
└─# ps -e -o pid,comm,policy | grep -v POLICY

    PID COMMAND         POL
      1 systemd         TS
      2 kthreadd        TS
      3 pool_workqueue_ TS
      4 kworker/R-rcu_g TS
      5 kworker/R-sync_ TS
      6 kworker/R-slub_ TS
      7 kworker/R-netns TS
     11 kworker/u8:0-ip TS
     12 kworker/R-mm_pe TS
     13 rcu_tasks_kthre TS
     14 rcu_tasks_rude_ TS
     15 rcu_tasks_trace TS
     16 ksoftirqd/0     TS
     17 rcu_preempt     TS
     18 rcu_exp_par_gp_ TS
     19 rcu_exp_gp_kthr TS
     20 migration/0     FF
     21 idle_inject/0   FF
     22 cpuhp/0         TS
     23 cpuhp/1         TS
     24 idle_inject/1   FF
     25 migration/1     FF
     26 ksoftirqd/1     TS
     33 kdevtmpfs       TS
     34 kworker/R-inet_ TS
     36 kauditd         TS
     37 khungtaskd      TS
     38 oom_reaper      TS
     40 kworker/R-write TS
     41 kcompactd0      TS
     42 ksmd            TS
     43 khugepaged      TS
     44 kworker/R-kinte TS
     45 kworker/R-kbloc TS
     46 kworker/R-blkcg TS
     47 irq/9-acpi      FF
     49 kworker/R-tpm_d TS
     50 kworker/R-edac- TS
     51 kworker/R-devfr TS
     53 kswapd0         TS
     61 kworker/R-kthro TS
     66 kworker/R-acpi_ TS
     67 kworker/R-mld   TS
     68 kworker/R-ipv6_ TS
     69 kworker/u8:1-ip TS
     74 kworker/R-kstrp TS
     78 kworker/u11:0   TS
     79 kworker/u12:0-t TS
     80 kworker/u13:0   TS
     85 kworker/R-crypt TS
    168 kworker/0:1H-kb TS
    225 kworker/R-ata_s TS
    228 scsi_eh_0       TS
    229 kworker/R-scsi_ TS
    230 scsi_eh_1       TS
    231 kworker/R-scsi_ TS
    234 scsi_eh_2       TS
    235 kworker/R-scsi_ TS
    247 irq/18-vmwgfx   FF
    248 kworker/R-ttm   TS
    284 jbd2/sda1-8     TS
    285 kworker/R-ext4- TS
    348 systemd-journal TS
    394 systemd-udevd   TS
    398 psimon          FF
    467 haveged         TS
    476 accounts-daemon TS
    478 dbus-daemon     TS
    483 polkitd         TS
    488 rsyslogd        TS
    489 systemd-logind  TS
    555 NetworkManager  TS
    598 ModemManager    TS
    607 kworker/R-rpcio TS
    609 kworker/R-xprti TS
    614 cron            TS
    661 VBoxService     TS
    816 gdm3            TS
    837 psimon          FF
    867 rtkit-daemon    TS
    951 colord          TS
   1041 upowerd         TS
   1210 pcscd           TS
   1325 wpa_supplicant  TS
   1354 power-profiles- TS
   1389 gdm-session-wor TS
   1397 systemd         TS
   1399 (sd-pam)        TS
   1422 pipewire-pulse  TS
   1423 dbus-daemon     TS
   1424 gnome-keyring-d TS
   1455 gdm-x-session   TS
   1458 Xorg            TS
   1486 gnome-session-b TS
   1570 VBoxClient      TS
   1572 VBoxClient      TS
   1585 VBoxClient      TS
   1586 VBoxClient      TS
   1593 VBoxClient      TS
   1594 VBoxClient      TS
   1626 at-spi-bus-laun TS
   1633 dbus-daemon     TS
   1645 VBoxClient      TS
   1646 VBoxClient      TS
   1662 gcr-ssh-agent   TS
   1663 gnome-session-c TS
   1675 gvfsd           TS
   1687 gvfsd-fuse      TS
   1696 gnome-session-b TS
   1722 gnome-shell     TS
   1754 dconf-service   TS
   1779 mutter-x11-fram TS
   1782 at-spi2-registr TS
   1798 xdg-desktop-por TS
   1803 xdg-document-po TS
   1807 xdg-permission- TS
   1814 fusermount3     TS
   1825 gnome-shell-cal TS
   1842 evolution-sourc TS
   1844 gjs             TS
   1862 ibus-daemon     TS
   1864 gsd-a11y-settin TS
   1867 gsd-color       TS
   1868 gsd-datetime    TS
   1869 gsd-housekeepin TS
   1870 gsd-keyboard    TS
   1877 evolution-alarm TS
   1883 gsd-media-keys  TS
   1892 blueman-applet  TS
   1897 gsd-disk-utilit TS
   1906 gsd-power       TS
   1908 gsd-print-notif TS
   1911 gsd-rfkill      TS
   1913 gsd-screensaver TS
   1916 gsd-sharing     TS
   1917 gsd-smartcard   TS
   1922 gsd-sound       TS
   1926 gsd-usb-protect TS
   1930 udisksd         TS
   1931 gsd-wacom       TS
   1935 gsd-xsettings   TS
   1960 kworker/0:2H    TS
   1999 gsd-printer     TS
   2027 ibus-memconf    TS
   2028 ibus-extension- TS
   2031 ibus-x11        TS
   2044 ibus-portal     TS
   2061 gjs             TS
   2068 goa-daemon      TS
   2103 evolution-calen TS
   2108 goa-identity-se TS
   2131 evolution-addre TS
   2157 gvfs-udisks2-vo TS
   2160 obexd           TS
   2170 gvfs-gphoto2-vo TS
   2175 gvfs-afc-volume TS
   2181 gvfs-goa-volume TS
   2186 gvfs-mtp-volume TS
   2191 ibus-engine-sim TS
   2201 tracker-miner-f IDL
   2202 xdg-desktop-por TS
   2255 xdg-desktop-por TS
   2272 Thunar          TS
   2299 kworker/u9:4-ev TS
   2337 gvfsd-trash     TS
   2357 gvfsd-metadata  TS
   2492 kworker/u10:0-e TS
   2493 kworker/1:0H    TS
   2630 fwupd           TS
   2670 kworker/u10:3-f TS
   2673 kworker/1:2-eve TS
   2681 kworker/u9:1-ev TS
   2682 kworker/u9:2-ev TS
   2820 kworker/u12:2   TS
   2865 kworker/u10:1-e TS
   2910 kworker/1:2H-kb TS
   2983 gjs             TS
   3001 kworker/0:0-eve TS
   3087 gnome-terminal- TS
   3100 zsh             TS
   3154 kworker/u10:2-e TS
   3212 sudo            TS
   3213 sudo            TS
   3214 su              TS
   3217 systemd         TS
   3218 (sd-pam)        TS
   3219 kworker/0:3-cgr TS
   3233 psimon          FF
   3238 pipewire        TS
   3239 pipewire        TS
   3240 zsh             TS
   3241 wireplumber     TS
   3242 pipewire-pulse  TS
   3244 dbus-daemon     TS
   3270 pipewire        TS
   3271 wireplumber     TS
   3306 kworker/1:0-ata TS
   3377 gvfsd           TS
   3383 gvfsd-fuse      TS
   3397 xdg-document-po TS
   3401 xdg-permission- TS
   3408 fusermount3     TS
   3446 ps              TS
   3447 grep            TS
```

TS = SCHED_OTHER, which is Linux’s default time-sharing policy.

SCHED_OTHER is implemented by the CFS (Completely Fair Scheduler).

You Also See FF and IDL

- FF = SCHED_FIFO (Real-time processes, rarely used unless explicitly assigned).
- IDL = SCHED_IDLE (for background tasks that should only run when CPU is otherwise idle).

But these are only used for a handful of special processes.

main scheduler policies are limited to the six listed below — these are defined in the Linux kernel and exposed via the POSIX API (sched_setscheduler, etc.):

| Policy Name        | Constant (C API) | `ps` Output | Scheduler Backend                  | Description                                         |
| ------------------ | ---------------- | ----------- | ---------------------------------- | --------------------------------------------------- |
| **SCHED_OTHER**    | `SCHED_OTHER`    | `TS`        | ✅ CFS (Completely Fair Scheduler) | Default for normal tasks                            |
| **SCHED_BATCH**    | `SCHED_BATCH`    | `TS`        | ✅ CFS                             | For non-interactive CPU-bound tasks (no preemption) |
| **SCHED_IDLE**     | `SCHED_IDLE`     | `IDL`       | ✅ CFS                             | Very low-priority tasks, run only when CPU is idle  |
| **SCHED_FIFO**     | `SCHED_FIFO`     | `FF`        | FIFO Real-Time Queue               | First-in-first-out real-time tasks, no time-slice   |
| **SCHED_RR**       | `SCHED_RR`       | `RR`        | Round-Robin Real-Time Queue        | Time-sliced real-time tasks, same priority          |
| **SCHED_DEADLINE** | `SCHED_DEADLINE` | `DLN`       | ✅ Deadline Scheduler              | Tasks with explicit deadlines; uses EDF + CBS       |

---

only these are used in modren kernels others are only bounded to theory or they are impractical

| Algorithm                                      | Status in Linux                                  | Notes                                                              |
| ---------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------------ |
| **Completely Fair Scheduler (CFS)**            | ✅ **Default scheduler** in Linux (since 2.6.23) | Used for normal tasks (`SCHED_OTHER`, `SCHED_BATCH`, `SCHED_IDLE`) |
| **O(1) Scheduler**                             | ❌ Obsolete                                      | Used before CFS (Linux 2.6.0 – 2.6.22)                             |
| **Round Robin (RR)**                           | ✅ For real-time tasks (`SCHED_RR`)              | Fixed priority, time-sliced                                        |
| **FIFO (First-In First-Out)**                  | ✅ For real-time tasks (`SCHED_FIFO`)            | Fixed priority, no time-slice                                      |
| **Earliest Deadline First (EDF)**              | ✅ Implemented as `SCHED_DEADLINE`               | Requires task deadline info                                        |
| **Rate-Monotonic (RMS)**                       | ❌ Not directly implemented                      | Can be approximated via fixed priority                             |
| **Shortest Job First (SJF)**                   | ❌ Not used in general-purpose OS                | Needs full job length info, impractical                            |
| **Multilevel Feedback Queue**                  | ❌ Not implemented in Linux                      | Used in some other OS (e.g., Windows NT)                           |
| **Lottery / Stride Scheduling**                | ❌ Not in mainline Linux                         | Research-oriented or academic OS use only                          |
| **Weighted Fair Queuing (WFQ)**                | ❌ Not CPU scheduler in Linux                    | Used in network packet scheduling, not CPU scheduling              |
| **Proportional Share Scheduling / Fair Share** | ❌ Not implemented as a scheduler                | Somewhat resembles `CFS` with cgroups or nice levels               |

### Summary

Only a **few are actually implemented in Linux**:

- **CFS** for normal processes.
- **FIFO**, **RR**, and **Deadline** for real-time processes.

The rest are either:

- Used in **research**, **academic**, or **specialized OSes**,
- Or are **infeasible** for general-purpose OS due to prediction problems (e.g., SJF), or
- Replaced by better, scalable alternatives (e.g., CFS over O(1)).

---

> [NOTE:]
> Priority Scheduling is implemented in Linux, but not as a standalone scheduler — it's integrated into the existing scheduling policies

---

<br>
<br>

We can change the CPU scheduling algorithm of a single process but to change the default CPU scheduling algorithm we need to compile the whole linux kernel

download the linux kernel source code and go to

```bash
linux/kernel/sched/fair.c
```

here you will see

| File                       | Purpose                                         |
| -------------------------- | ----------------------------------------------- |
| `kernel/sched/core.c`      | Main scheduler logic (common to all schedulers) |
| `kernel/sched/fair.c`      | CFS logic                                       |
| `kernel/sched/rt.c`        | Real-time scheduler (SCHED_RR, SCHED_FIFO)      |
| `kernel/sched/deadline.c`  | Earliest Deadline First (SCHED_DEADLINE)        |
| `kernel/sched/idle.c`      | Idle task scheduler                             |
| `kernel/sched/stop_task.c` | Used during CPU hotplug and stop machine tasks  |
| `include/linux/sched.h`    | Task and scheduling structure definitions       |
