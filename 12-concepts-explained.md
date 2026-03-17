# 12. Concepts Explained

Definitions and “why we need it” for terms used in the Meta PE guide. Use this when the guide mentions a concept without full explanation.

---

## NFS (Network File System)

**What it is:**
- Protocol and software to share directories over the network.
- Server **exports** directories; clients **mount** them and access files as if local.

**Why we need it:**
- Shared storage across many hosts (e.g. shared home dirs, application data, build trees) without copying files.
- Centralized storage that multiple machines can read/write.

**How it works:**
- Server runs **nfsd** and **mountd**; exports in **/etc/exports**.
- Client: `mount -t nfs server:/exported/path /local/mountpoint`.
- Uses **RPC** for initial connection; then file operations over the network.
- **NFSv3** — UDP or TCP. **NFSv4** — stateful, often single port.

**Why it appears in troubleshooting:**
- NFS is I/O over the network. If server or network is slow/down, clients block in **D state** (uninterruptible sleep) waiting for NFS replies.
- Raises load; **ls** or other commands can hang.
- See **08-troubleshooting.md** (NFS section).

---

## VFS (Virtual File System)

**What it is:**
- Kernel layer that provides one common interface for all filesystem types (ext4, XFS, NFS, tmpfs, etc.).
- Applications call **open()**, **read()**, **write()**, **close()**; VFS routes to the right driver.

**Why we need it:**
- Programs don’t need to know whether a path is on ext4, NFS, or something else.
- One API for every filesystem; kernel hides differences (local disk vs network, different on-disk layouts).

**Key objects:**
- **Superblock** — per mounted FS: type, block size, mount info.
- **Inode** — file/dir metadata.
- **Dentry** — name → inode; **dcache** caches for path lookup.
- **File** — per open file: offset, mode, link to dentry/inode.
- **Relationship:** File → Dentry → Inode → data blocks.

---

## initrd / initramfs (Initial RAM Disk / Initial RAM File System)

**What it is:**
- Small filesystem loaded into RAM with the kernel at boot.
- **initrd** — usually a disk image (e.g. ext2) used as a RAM disk.
- **initramfs** — compressed **cpio** archive unpacked into tmpfs; modern, preferred.

**Why we need it:**
- Kernel needs drivers and tools to find and mount the **real** root (e.g. RAID, LVM, network, encrypted disk).
- Those may not be built into the kernel. initramfs holds drivers and minimal **/init** script that mounts real root and runs **/sbin/init**.
- Without it, kernel might not be able to mount root.

**Difference:**
- initrd = block device in RAM with normal FS.
- initramfs = cpio archive; no block layer; simpler and usually faster.
- After root is mounted, initramfs can be discarded.

---

## Runlevels (traditional) and systemd targets

**What they are:**
- Single number (runlevel) or named target that defines “what the system is for”: halt, single-user, multi-user, GUI, etc.

**Why we need them:**
- Boot into different modes (recovery vs normal, server vs desktop) by starting the right set of services.

**Runlevels:**
- **0** = halt
- **1** = single-user (maintenance)
- **3** = multi-user CLI
- **5** = multi-user with GUI
- **6** = reboot
- Programs in **/etc/rc.d/rc*.d/**.
- **who -r** — current runlevel; **init N** — switch.

**systemd:** Replaces runlevels with **targets** (e.g. multi-user.target, graphical.target, rescue.target, emergency.target). **systemctl get-default** / **set-default**.

---

## CFS (Completely Fair Scheduler)

**What it is:**
- Default Linux scheduler for normal (non–real-time) processes.
- Tries to give each runnable process a “fair” share of CPU over time.

**Why we need it:**
- So one process doesn’t starve others and interactive tasks stay responsive.
- **vruntime** (virtual runtime): advances when process runs, weighted by **nice** (lower nice = higher priority = vruntime advances more slowly).
- Scheduler always picks process with **smallest vruntime** (red-black tree).
- Timeslices are **dynamic**, not fixed.

**Contrast:** **SCHED_FIFO** and **SCHED_RR** (real-time) can starve normal CFS processes if misused.

---

## DMA (Direct Memory Access) and ZONE_DMA

**What DMA is:**
- Hardware (e.g. disk controller, NIC) reading/writing RAM **without** the CPU copying each byte.
- CPU sets up transfer (address, size); device does the rest; CPU can do other work. Reduces load and latency.

**Why ZONE_DMA exists:**
- Older devices (e.g. ISA) could only address first 16 MB of physical memory.
- Kernel reserves **ZONE_DMA** (and **ZONE_DMA32** on 64-bit) so DMA buffers are in address ranges those devices can use.
- **ZONE_NORMAL** — normal kernel RAM. **ZONE_HIGHMEM** (mainly 32-bit) — RAM not permanently mapped in kernel space.

---

## Slab allocator / Slab layer

**What it is:**
- Kernel subsystem that allocates/frees **kernel objects** (e.g. task_struct, inodes, dentries) in fixed-size, pre-initialized chunks instead of generic page allocator.

**Why we need it:**
- Kernel creates/destroys many small, same-sized structures. Page allocator for each would cause **fragmentation** and extra work.
- **Caches** (one per object type) → **slabs** (contiguous pages) → **objects**. Fast alloc/free; less fragmentation; better cache locality.
- **/proc/slabinfo**, **slabtop**.

---

## Page cache and buffer cache (buffers)

**Page cache:**
- Kernel cache of **file contents** in RAM. read()/write() go through it. Reads from RAM when possible; writes to RAM first (dirty), flushed by **pdflush**/ **kworker** or **sync()**/ **fsync()**. Minimizes disk I/O.

**Buffer cache / buffers:**
- Historically separate cache for **block device** metadata (superblocks, inodes, directory blocks) and raw block I/O.
- Modern kernels: largely **unified** with page cache. In **free**, “buffers” = block device metadata; “cached” = page cache for file data. Both reclaimable under memory pressure.

---

## RPC (Remote Procedure Call)

**What it is:**
- One program calls a “function” that runs on another machine. Caller sends request (procedure ID + arguments); other side runs procedure and returns result. Caller waits as if local call.

**Why it matters for NFS:**
- NFS uses RPC for initial mount and some control operations. **rpcbind** (portmap) maps RPC service names to ports. **rpcinfo -p** lists RPC services. If NFS mount fails, check **rpcinfo -p <server>**; if RPC or **nfsd**/ **mountd** aren’t running, mount will fail.

---

## BSS (Block Started by Symbol)

**What it is:**
- Segment in executable/object file that holds **uninitialized** global and static variables. File doesn’t store contents (all zero); only how much space needed.

**Why we need it:**
- Saves space in binary; at load time OS zeros that range. In ELF, **.bss** has **Memsz** often larger than **Filesz**; difference zero-initialized at load.

---

## TLB (Translation Lookaside Buffer)

**What it is:**
- Small, fast cache on CPU that holds recent **virtual → physical** page mappings. CPU checks TLB first; if hit, gets physical address without walking page tables.

**Why it matters:**
- Page table walks cost several memory accesses. TLB makes normal access fast. **TLB miss** triggers walk (may cause page fault). **Huge pages** reduce TLB misses (one entry covers more address space).

---

## NUMA (Non-Uniform Memory Access)

**What it is:**
- On multi-socket systems, memory is attached to specific sockets. “Local” memory faster than “remote” (another socket’s). Kernel/apps can try to keep process and data on same NUMA node.

**Why it matters:**
- For performance-sensitive or memory-heavy workloads, binding processes to CPUs and memory from same node can reduce latency and improve throughput. **lscpu** shows topology; **numactl --hardware**; **numactl** can bind processes and memory policy.

---

## CNI (Container Network Interface)

**What it is:**
- Standard (spec + plugins) for how container runtimes (e.g. Kubernetes) create and attach **network interfaces** for containers/pods. Runtime calls CNI plugin with config; plugin creates veth pairs, connects to bridge/overlay, returns container IP.

**Why we need it:**
- Different orchestrators and runtimes can use same or pluggable networking (Calico, Flannel, Cilium) without each implementing everything.

---

## Seccomp (Secure Computing Mode)

**What it is:**
- Kernel feature that **restricts which system calls** a process can make. Process (or parent) installs filter (e.g. BPF); kernel blocks or allows syscalls based on filter.

**Why we need it:**
- In containers/sandboxes, reduce attack surface: if process is compromised, it still can’t call dangerous syscalls. Containers often run with default seccomp profile that disallows many unneeded syscalls.

---

## Capabilities

**What they are:**
- Splitting root’s all-or-nothing privilege into separate **capabilities** (e.g. CAP_NET_BIND_SERVICE for ports < 1024, CAP_SYS_ADMIN). Process can have subset instead of full root.

**Why we need them:**
- Containers or services can do only what they need (e.g. bind to 80) without full root. If compromised, attacker doesn’t get all privileges. **getcap**/ **setcap** on files; capability sets per process.

---

## AppArmor and SELinux (MAC)

**What they are:**
- **Mandatory Access Control (MAC)**. Kernel enforces policy (from config) on which subjects (processes) can access which objects (files, ports, etc.). Unlike Unix DAC (owner/group/other), policy is central and can’t be overridden by process owner.

**Why we need them:**
- Confine applications and containers: even as root, process can only do what policy allows. **AppArmor** — path-based profiles. **SELinux** — labels (contexts) on files and processes. Denials in **journalctl**, **ausearch**; misconfiguration can block legitimate access.

---

## sk_buff (socket buffer)

**What it is:**
- Kernel’s main structure for a **network packet** (or piece) as it moves through the stack. Holds packet data, headers, metadata (device, protocol state). Often abbreviated **skb**.

**Why it matters:**
- When guide says “interrupt handler copies packets into memory (e.g. sk_buff)”, driver’s top half allocates skbs, copies from NIC, passes to network stack. Delays in that copy → NIC buffer overflow → **dropped packets**.

---

## IDT (Interrupt Descriptor Table) and TSS (Task State Segment)

**IDT:**
- Table in memory; CPU uses it to find **handler** for each interrupt/exception number. **IDTR** register points to IDT. On interrupt (hardware IRQ, **int 0x80**, or CPU exception), CPU looks up number and jumps to handler (e.g. syscall entry, device driver ISR).

**TSS:**
- Structure holding **kernel stack** pointer and other state for a CPU. On switch from user to kernel mode (e.g. syscall or interrupt), CPU loads kernel stack pointer from TSS. Linux often uses one TSS per CPU.

---

## Other terms (brief)

- **CPL (Current Privilege Level):** In code segment selector; 0 = kernel, 3 = user. Syscall/exception → CPL 0; **sysret**/ **iret** restores 3.
- **GDT (Global Descriptor Table):** Describes code/data segments; **gdtr** points to it. Real Mode → Protected Mode uses GDT and CR0.
- **NIS (Network Information Service):** Old directory service for users/groups/hosts. Guide mentions **nisping -u** in NFS troubleshooting (“local name service” on client).
- **MegaCLI, storcli:** Vendor tools for **hardware RAID** controllers (e.g. LSI/Broadcom). Used when RAID is on dedicated card, not software **mdadm**.
