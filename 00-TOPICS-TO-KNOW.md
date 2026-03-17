# Linux Topics to Know (Checklist)

Notes are merged into **12 topic files** (01–12). Use this list with **01-fundamentals.md** through **12-concepts-explained.md**.

**Interview focus:** Text manipulation, I/O, automation, interfacing with systems. See **11-pe-interview-skills.md**.

---

## Linux fundamentals

- [[Linux architecture]] · [[Shell]] · [[Kernel]] · [[User space]] · [[Filesystem]] · [[Users & groups]] · [[Process]] · [[Threads]] · [[Signals]] · [[Environment variables]] · [[Bash commands]] · [[Kernel synchronization]] · [[NFS]]

*Notes: 01 (fundamentals), 03 (boot & kernel).*

---

## Process & system management

- [[PID]] · [[Daemon]] · [[Process scheduling]] · [[Context switching]] · [[systemd]] · [[Init system]] · [[Process states]] · [[Signals]] · [[Process control block]]

*Notes: 04 (processes & memory), 05 (services & systemd).*

---

## Memory management

- [[Virtual memory]] · [[Paging]] · [[Swap]] · [[Heap]] · [[Stack]] · [[Memory leak]] · [[OOM killer]] · [[Page cache]]

*Notes: 04 (processes & memory), 07 (performance).*

---

## Filesystem & storage

- [[Inode]] · [[File descriptor]] · [[Hard link]] · [[Symbolic link]] · [[Mount]] · [[VFS]] · [[I/O schedulers]] · [[RAID]]

*Notes: 02 (filesystem & storage).*

---

## Boot process

- [[BIOS]] · [[UEFI]] · [[POST]] · [[Bootloader]] · [[GRUB]] · [[Kernel initialization]] · [[initramfs]] · [[systemd startup]]

*Notes: 03 (boot & kernel), 05 (systemd).*

---

## System performance & debugging

- [[CPU usage]] · [[Memory usage]] · [[Disk I/O]] · [[Load average]] · [[I/O wait]] · [[USE method]] · [[RED method]]

*Notes: 07 (performance & logs), 10 (commands reference).*

---

## Networking

- [[TCP]] · [[UDP]] · [[DNS]] · [[Routing]] · [[Ports]]

*Notes: 06 (networking).*

---

## Scaling & troubleshooting

- [[Horizontal scaling]] · [[Vertical scaling]] · [[Caching]] · [[OOM]] · [[Database slow]] · [[NFS troubleshooting]]

*Notes: 08 (troubleshooting).*

---

## Advanced

- [[Containers]] · [[Namespaces]] · [[Cgroups]] · [[Virtualization]] · [[ELF]]

*Notes: 09 (advanced topics).*

---

## Reference

- **10-commands-reference.md** — Essential commands, ls -l *.c flow, Ctrl+C, FD kernel tables, ELF, strace/ltrace/gdb.
- **11-pe-interview-skills.md** — Skill mapping and checklist.
- **12-concepts-explained.md** — **Definitions and “why we need it”** for every concept the guide mentions: NFS, VFS, initrd/initramfs, runlevels, CFS, DMA, slab, page cache, RPC, BSS, TLB, NUMA, CNI, Seccomp, Capabilities, AppArmor/SELinux, sk_buff, IDT/TSS, and more.
