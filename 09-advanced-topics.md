# 9. Advanced Topics

*Merged: Containers, virtualization, ELF. See **12-concepts-explained.md** for CNI, Seccomp, Capabilities, AppArmor/SELinux, NFS.*

---

## Linux containers (guide)

**What**
- Lightweight OS-level virtualization; apps in isolated environments on **shared kernel**.
- **Stack:** Container apps → Runtime (Docker, Podman) → Engine (containerd, CRI-O) → Kernel (namespaces, cgroups, etc.).

**Namespaces** — Isolate resources
| Namespace | Isolates |
|-----------|----------|
| **PID** | Process IDs; first in new namespace = PID 1 |
| **Mount** | Filesystem mount points |
| **UTS** | Hostname, domain |
| **IPC** | System V IPC, POSIX msg queues |
| **User** | UID/GID; root in namespace ≠ root on host |
| **Network** | Devices, IPs, routing, ports |
| **Time** | System time |

- **Inspect:** **ls -la /proc/&lt;PID&gt;/ns/**; **nsenter -t &lt;PID&gt; -n -p** (enter network + PID namespace).

**Cgroups** — Limit, account, isolate CPU, memory, disk I/O, network
- **Features:** resource limits, prioritization, accounting, control (freeze/stop).
- **Path:** **/sys/fs/cgroup/** (e.g. memory/, cpu/).
- **Process cgroup:** **cat /proc/&lt;PID&gt;/cgroup**; limits e.g. **memory.limit_in_bytes**.
- Used in Kubernetes for pod requests/limits.

**Security**
- **Capabilities** — Split root into units; containers get only what they need (see **12-concepts-explained.md**).
- **Seccomp** — Filter allowed syscalls (see **12**).
- **AppArmor / SELinux** — MAC to confine containers (see **12**).
- **CNI** — Standard for container networking; plugins (Calico, Flannel, Cilium) implement it (see **12**).

---

## Virtualization — VMs (guide)

**Structure**
- Guest app → Guest OS → **Hypervisor (VMM)** → Host OS (Type 2) or hardware (Type 1) → hardware.
- **Type 1 (bare-metal):** ESXi, Xen, Hyper-V.
- **Type 2 (hosted):** VirtualBox, VMware Workstation/Fusion.

**CPU**
- **Trap-and-emulate** — guest privileged instructions trapped by VMM, emulated.
- **VT-x / AMD-V** make this efficient.

**Memory**
- VMM maps **GVA → GPA → HPA**.
- **Shadow page tables** — VMM maintains GVA→HPA; when guest changes page tables (e.g. CR3), VMM intercepts and updates shadow tables.
- **EPT (Intel) / NPT (AMD)** — hardware does GVA→HPA for better performance.

**Containers vs VMs**
- Containers virtualize OS (share kernel); no hypervisor; lightweight.
- VMs virtualize hardware (own kernel per VM); need hypervisor; stronger isolation; more overhead.
- Density: more containers per host than VMs.

---

## ELF (guide)

**Structure**
1. **ELF header** — magic, class (32/64), type (exec, relocatable, shared), machine, **entry point**.
2. **Program header table (segments)** — load time; PT_LOAD segments (.text, .data, .bss); Vaddr, Filesz, Memsz (Memsz &gt; Filesz for BSS).
3. **Section header table** — link time; .text, .data, .bss, .rodata, symbol table, relocations.
4. Data.

**Loading**
- Kernel reads ELF and program headers; for each PT_LOAD, maps segment at Vaddr with permissions.
- If **dynamically linked**, runs **interpreter** (dynamic linker), which loads shared libs (PT_DYNAMIC) and resolves symbols; sets initial stack; jumps to entry (or linker’s).

**Static vs dynamic linking**
- **Static** — All libs in executable at link time; self-contained; larger; lib bug = recompile all.
- **Dynamic** — Lib names in executable; linking at load/first use; smaller binary; shared libs in RAM; libs updatable; risk of version conflicts.
- **Linker** — combines object files into executable/library; resolves symbols.
- **Loader** — OS loads program into memory and prepares execution.

---

## NFS, SELinux (brief)

- **NFS** — Full explanation and troubleshooting in **12-concepts-explained.md** and **08-troubleshooting.md**.
- **SELinux/AppArmor** — MAC; check denials in journalctl/ausearch.
