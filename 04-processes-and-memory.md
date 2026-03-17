# 4. Processes and Memory

*Merged: Processes, memory management. See **12-concepts-explained.md** for CFS, DMA, slab, page cache, BSS, TLB, NUMA.*

---

## Process Control Block (PCB / task_struct)

Each process has a PCB with:
- **PID** and **PPID**
- **Process state** (R/S/D/Z/T/X)
- **CPU registers** (including **program counter**)
- **Memory** limits and page tables
- **Open files / file descriptors**
- **Signal handlers** and pending signals
- **Scheduling** (priority, nice)
- **Resource usage**
- **Pointers** to parent and child PCBs

---

## Process states (guide)

| State | Meaning |
|-------|---------|
| **R** | Running or runnable (on CPU or in run queue) |
| **S** | Interruptible sleep (waiting for event; can be interrupted) |
| **D** | Uninterruptible sleep (waiting for I/O; **cannot be killed** by SIGKILL; needs reboot or I/O completion; **counts in load average**) |
| **T** | Stopped (e.g. SIGSTOP or debugger) |
| **Z** | Zombie (exited, parent has not reaped with wait()); no CPU/memory except process table entry |
| **X** | Dead (being destroyed; rarely seen) |

**States in load average:** R and D.

---

## fork(), CoW, exec(), vfork(), clone()

**fork()**
- Duplicates caller; child gets copy of address space (code, data, heap, shared libs, stack).
- **Same open FDs** (point to same file table entry); new PID; PPID = parent.
- **Pending signals not inherited**.
- Returns: parent = child PID, child = 0, failure = -1.
- Implemented via **clone()** → **do_fork()**: dup_task_struct (stack, thread_info, task_struct), resource check, copy_flags (PF_FORKNOEXEC etc.), alloc_pid, wake child.

**Copy-on-Write (CoW)**
- Parent and child share pages (read-only); on first write, page fault → kernel copies page for writer.
- Saves work when child soon **exec()**s.

**exec()** (e.g. execve())
- Replaces process image with new program; new address space; stack/heap/data replaced.
- Same PID.

**vfork()**
- Child shares parent address space (no copy); **parent blocked** until child **exec()** or **exit()**; child runs first.
- Child must not write (can corrupt). Efficient when child immediately execs.

**clone()**
- Underlying call for fork/vfork/threads.
- Flags: **CLONE_VM** (address space), **CLONE_FS**, **CLONE_FILES**, **CLONE_SIGHAND**.
- fork ≈ clone(SIGCHLD, 0); vfork ≈ clone(CLONE_VFORK | CLONE_VM | SIGCHLD, 0); thread ≈ clone(CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND, 0).

**wait() / waitpid() / wait4()**
- Parent waits for child state change (e.g. exit); gets exit status.
- **wait4()** can return resource usage.

**exit()**
- Terminates process; process becomes **zombie** until parent **wait()**s.
- **Zombie cleanup:** kill -CHLD parent_pid or kill parent → **init** adopts and reaps.
- **Orphan** — parent died first; adopted by init (PID 1); init reaps so orphans don’t stay zombies.

---

## Init process (PID 1)

- First process started by kernel; ancestor of all others.
- Initializes system (runlevels/targets); **adopts orphans**.
- Traditional: SysV init, /etc/inittab. Modern: systemd.

---

## IPC (guide)

| Mechanism | Notes |
|-----------|--------|
| **Pipes** | pipe() returns read/write FDs; unidirectional; parent–child |
| **Named pipes (FIFO)** | mkfifo; has path; unrelated processes |
| **Shared memory** | Fastest; needs sync (semaphores/mutexes) |
| **Message queues** | Kernel queue; structured messages, priorities |
| **Sockets** | Unix domain (local path) or TCP/UDP (network); full-duplex |
| **Signals** | Async; stored in PCB; on delivery process interrupted → kernel → handler |

**Signals**
- **SIGKILL (9)** and **SIGSTOP (17/19)** cannot be caught, blocked, or ignored.
- Common: **SIGTERM (15)**, **SIGINT (2, Ctrl+C)**, **SIGHUP (1)**, **SIGCHLD**, **SIGPIPE**, **SIGUSR1/2**.
- **kill -HUP &lt;PID&gt;** often reloads config.

---

## Nice value and scheduling

- **Nice** — -20 (highest priority) to +19 (lowest).
- **nice -n &lt;value&gt; ./cmd**; **renice -n &lt;value&gt; -p &lt;PID&gt;**.
- In **CFS** (see **12-concepts-explained.md**), nice influences **vruntime** progression.

**CFS**
- vruntime per process; scheduler runs process with **smallest vruntime**; red-black tree; **dynamic** timeslices.

**Real-time**
- **SCHED_FIFO** — runs until block/yield or preempted by higher-priority RT.
- **SCHED_RR** — like FIFO but fixed timeslice per priority.
- **chrt** for RT policies.

---

## Threads and kernel threads

- In Linux, threads are **processes** sharing resources (address space, open files, signal handlers) via **clone()** flags.
- **Kernel threads** — run only in kernel; **mm = NULL**; no user VAS.
- Examples: ksoftirqd, kswapd; created only by other kernel threads; all forked from **kthreadd** (PID 2); kthreadd PPID = 0 (spawned by kernel).
- **ps -ef** shows them.

---

## Context switching (guide)

1. Save current process context (registers, PC, stack) into PCB.
2. Move PCB to relevant queue (ready, blocked, I/O).
3. Scheduler picks next from ready queue.
4. Load new process context from PCB.
5. Update page tables (maybe TLB flush).
6. Resume new process.

**Costs:** Register save/restore, cache pollution, TLB flush, memory barriers.

**Monitoring:** **vmstat 1** (cs = context switches/sec), **pidstat -w 1** (voluntary/involuntary), **perf record -e sched:sched_switch -g**.

**System call** = mode switch (user↔kernel); **context switch** only if process blocks (e.g. I/O, mutex).

---

## Memory: MMU and VAS (guide)

**MMU**
- Hardware that translates **virtual → physical** using page tables; enforces protection.
- CPU always uses virtual addresses.
- Advantages: efficient RAM use, less fragmentation, isolation.
- (See **12-concepts-explained.md** for TLB.)

**VAS layout**
- **Kernel space** (top; user inaccessible)
- **Stack** (grows down); gap
- **mmap** (shared libs, mmap files)
- **Heap** (brk/sbrk, malloc)
- **BSS** (uninit global/static; zeroed)
- **Data** (init global/static)
- **Text** (code; read-only)

**brk()/sbrk()** — change program break (heap size).
**mmap()** — map file or anonymous region; **munmap()** — unmap.

---

## Paging and page faults (guide)

- **Pages** (e.g. 4 KB); **page frames** in physical RAM.
- **Page tables** map **VPN → PFN**. **TLB** caches translations.

**Address translation:** CPU → virtual address → TLB lookup → hit: use PFN; miss: page table walk → if not in memory or invalid → **page fault**.

**Page fault types:**
- **Minor** — page in RAM but not in page table (no disk I/O).
- **Major** — must load from disk (swap/file).
- **Segmentation/Protection** — invalid access → SIGSEGV.
- **Kernel memory** — generally not pageable (kernel allocates at init; violations → oops).

---

## Kernel memory zones (guide)

| Zone | Purpose |
|------|---------|
| **ZONE_DMA** | For DMA by old devices (&lt;16 MB) |
| **ZONE_DMA32** | 32-bit devices on 64-bit (≤4 GB) |
| **ZONE_NORMAL** | Normal kernel pages; permanently mapped |
| **ZONE_HIGHMEM** | 32-bit with &gt;1 GB; not permanently mapped |

(See **12-concepts-explained.md** for DMA.)

---

## Slab allocator (guide)

- **Caches** (per object type, e.g. PCBs, inodes) → **slabs** (contiguous blocks) → **objects** (pre-initialized).
- **Slab states:** Full, Partial, Empty.
- **Process:** kernel requests object → allocator gives from partial/empty slab; if none, new slab from pages; on free, object marked unused; free interface used when memory low.
- **Benefits:** less fragmentation, fast alloc/free, cache locality.
- **/proc/slabinfo**, **slabtop**. (See **12-concepts-explained.md**.)

---

## Swap (guide)

**Why:** When RAM is full, kernel swaps out less-used pages to disk to free RAM; startup-only pages can be swapped.

**Forms:** **Swap partition** (dedicated) or **swap file** (in FS).

**Pros:** Extra virtual memory, hibernation. **Cons:** Slow (disk), fixed size for partition, wear.

**kswapd:**
1. Kernel sees memory pressure.
2. kswapd runs.
3. LRU-like choice of pages.
4. Anonymous pages → swap.
5. File-backed: clean = discard, dirty = write back then reclaim.

**Swappiness** (vm.swappiness 0–100, default 60): lower = prefer dropping page cache over swapping; 0 = no anonymous swap unless OOM.

**Commands:** **swapon -s**, **cat /proc/swaps**, **free -h**.

**Create swap file:** dd if=/dev/zero of=/swapfile bs=1M count=1024, chmod 600 /swapfile, mkswap /swapfile, swapon /swapfile; add to /etc/fstab.

**Thrashing** — system spends most time paging; little real work.

---

## OOM Killer (guide)

- When memory (including swap) is exhausted and kernel cannot free enough, **OOM killer** runs.
- Chooses process by **oom_score** (/proc/&lt;PID&gt;/oom_score).
- **Factors:** memory use, CPU time, privileges (root less likely), **oom_score_adj** (-1000 to +1000; **-1000** = never kill).

**Dealing with high memory**
- **Temporary:** raise nice, **kill -SIGSTOP** unneeded processes.
- **Permanent:** **ulimit**/cgroups/limits.conf, find leaks (valgrind, ltrace; top trend), tune app or add RAM.

**Monitoring:** free, vmstat, top/htop; **dmesg | egrep -i 'killed process'**; **/proc/sys/vm/panic_on_oom** (0 = don’t panic). Scripts can log **/proc/*/status** for intermittent issues.

---

## Memory hierarchy (guide)

Registers (&lt;1 ns) → L1 (1–2 ns) → L2 (3–10 ns) → L3 (10–20 ns) → RAM (50–100 ns) → SSD (0.1–1 ms) → HDD (5–10 ms). Cache: tag, data block, flags; write-through/write-back; LRU eviction.

---

## Key VM parameters and tools (guide)

**sysctl vm.\*** or **/proc/sys/vm/:** **vm.swappiness**, **vm.dirty_ratio**, **vm.dirty_background_ratio**, **vm.overcommit_memory** (0/1/2), **vm.overcommit_ratio**, **vm.min_free_kbytes**, **vm.zone_reclaim_mode**.

**Pressure:** high si/so (vmstat), many D, OOM, high page faults.

**Tools:** **pmap -x &lt;PID&gt;**, **/proc/&lt;PID&gt;/maps**, **/proc/&lt;PID&gt;/smaps**, **/proc/meminfo**, **/proc/vmstat**, **/proc/buddyinfo**, **/proc/slabinfo**, **/proc/pagetypeinfo**; logs for kswapd/oom; **dmesg | grep -i "allocation failure"**.

**Huge pages:** **/proc/sys/vm/nr_hugepages**; mount hugetlbfs; **mmap** with MAP_HUGETLB; **cat /proc/meminfo | grep Huge**.
