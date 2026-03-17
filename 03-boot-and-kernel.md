# 3. Boot and Kernel

*Merged: Hardware/boot/runlevels, Linux kernel. See **12-concepts-explained.md** for initrd/initramfs, runlevels, Real/Protected mode, IDT, TSS.*

---

## Boot process (complete sequence from guide)

**Phase 1 — BIOS/UEFI**
- Power → **POST** (checks memory, CPU, motherboard, screen, keyboard; BIOS on ROM).
- Hardware detection → boot device selection.
- Load **MBR** (first 512 bytes) from bootable media.

**Phase 2 — MBR and boot loader**
- **MBR layout:**
  - First 446 bytes = primary boot loader code.
  - Next 64 bytes = partition table (4 entries × 16 bytes).
  - Last 2 bytes = boot signature **0x55AA**.
- MBR at first sector of disk (e.g. /dev/sda).
- **GRUB Stage 1** (in MBR, limited) → optional **Stage 1.5** (filesystem drivers) → **Stage 2** (full GRUB from /boot, reads **/boot/grub/grub.cfg**, menu for OS/kernel choice).
- Loads **vmlinuz** (compressed kernel) and **initrd/initramfs** into RAM → passes control to kernel.

**Phase 3 — Kernel**
1. Self-extraction.
2. Hardware detection (CPU, memory, PCI).
3. Memory management (page tables, virtual memory).
4. **initrd/initramfs** — temporary root in RAM with programs/drivers to mount real root.
5. Load storage, network, filesystem drivers.
6. Mount **real root** from disk; then initramfs can be cleared.
- **initrd** = RAM disk image with a traditional FS.
- **initramfs** = compressed cpio archive; generally more efficient (see **12-concepts-explained.md**).

**Phase 4 — Init (PID 1)**
- First userspace process; mounts filesystems, starts services, user login.
- Traditional: **SysV init**, **/etc/inittab**, **runlevels** (see below).
- Modern: **systemd** with **targets**.
- **/etc/fstab** defines mount points and options; **mount -a** mounts all in fstab except **noauto**.
- Special FS: **/proc**, **/sys**, **/dev** also mounted.

**Runlevels (traditional)**
- 0 = halt
- 1 = single-user (maintenance)
- 2 = multi-user no network
- 3 = multi-user with network (CLI)
- 4 = user-definable
- 5 = multi-user with network (GUI)
- 6 = reboot
- Programs in **/etc/rc.d/rc*.d/**.
- **who -r** = current runlevel; **init &lt;N&gt;** = set runlevel.

**Real mode vs Protected mode**
- **Real mode** — BIOS and early boot; 16-bit instructions, 16-bit registers, 1 MB address limit.
- **Protected mode** — after boot; **GDT** loaded into **gdtr**, bit set in **CR0** (see **12-concepts-explained.md**).

---

## Boot troubleshooting (guide scenarios)

**Scenario 1 — System hangs during boot**
- Remove **quiet** and **splash** from GRUB kernel parameters.
- Boot into single-user (**single** or **systemd.unit=rescue.target**) or **systemd.unit=emergency.target**.
- Check logs: **journalctl -xb**.
- Common causes: corrupted filesystem (**fsck**), hardware failure (**dmesg**), driver/config errors.

**Scenario 2 — Kernel panic**
- Triggers: missing/corrupted initrd/initramfs, wrong **root=** in GRUB, hardware incompatibility/failure, corrupted kernel image.
- Recovery: boot from live USB/rescue media → mount root → **chroot** into system → reinstall kernel → rebuild initramfs (**update-initramfs -u**) → **update-grub**.

**Scenario 3 — GRUB rescue**
- Causes: corrupted GRUB config, missing kernel files, wrong partition refs.
- Manual boot: **ls** (list partitions), **set root=(hdX,msdosY)**, **linux /boot/vmlinuz... root=/dev/sdXY**, **initrd /boot/initrd.img...**, **boot**.

**Boot performance**
- **systemd-analyze** (total time), **systemd-analyze blame** (slowest services), **systemd-analyze critical-chain**, **systemd-analyze plot > boot.svg** (timeline).
- Optimizations: disable unnecessary services, smaller initramfs, SSD for boot, reduce GRUB timeout, enable parallel service startup.

---

## Linux kernel (full guide detail)

**System calls**
- Only legal entry from user space (apart from exceptions/traps).
- Why: abstract hardware, security/stability (kernel checks permissions).
- Return: long; 0 = success, negative = errno.

**How (x86):**
1. User puts syscall number (e.g. %eax/%rax) and args in registers.
2. **int 0x80** or **syscall** → trap.
3. Mode switch: CPL 3→0; user context (ESP, EIP, CS, SS, eflags) saved on kernel stack (from **TSS**).
4. IDT gives handler.
5. **sys_call_table**[number] runs kernel function.
6. Result in %eax/%rax; **iret**/**sysret** restores user context.
- Why not call kernel directly? Different stacks (args in registers); kernel in different segment (privilege/protection).

**Adding a syscall:** Define in kernel, add to syscall table (e.g. arch/.../syscall_64.tbl), declare in syscalls.h, add to Makefile, recompile.

**Interrupts vs exceptions**
- **Interrupts** — async, from hardware (NIC, disk).
- **Exceptions** — sync:
  - **Fault** (page fault, div by zero; IP at faulting instruction).
  - **Trap** (syscall, breakpoint; IP after trap).
  - **Abort** (severe).
- **IDT** (base in IDTR) maps number → **ISR**.

**Interrupt context**
- ISR cannot sleep (no process to schedule); often shares stack with interrupted process.
- **Top half** — ack hardware, copy data, schedule bottom half, exit fast; delay = buffer overrun (e.g. dropped packets).
- **Bottom half:** **softirqs** (static, high-frequency; networking, block), **tasklets** (on top of softirqs; one per type at a time), **work queues** (kernel threads; can sleep).
- Use: top half = quick; softirq = timing-critical; tasklet = simpler; work queue = needs sleep.
- **ksoftirqd** — per-CPU thread (nice 19) drains softirqs so they don’t starve user processes.

**Kernel synchronization**
- **Atomic ops** (atomic_inc, test_and_set_bit).
- **Spinlock** — busy-wait; short critical section; disable local interrupts before acquire (same CPU); cannot sleep.
- **Semaphore** — sleeping lock; longer sections; not in interrupt context.
- **Mutex** — like binary semaphore; PID of waiter in structure.
- Choice: short/no sleep → spinlock; long/can sleep → semaphore/mutex; interrupt context → spinlock only.

**Other**
- **/proc** — Virtual FS; files generated on read (e.g. /proc/cpuinfo, meminfo, interrupts, /proc/&lt;PID&gt;/).
- **Kernel oops** — deviation from correct behavior; trace + register dump; may kill process; kernel unstable.
- **Kernel panic** — fatal; halt or reboot; may write vmcore.
