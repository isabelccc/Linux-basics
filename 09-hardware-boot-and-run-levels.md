# 9. Hardware, Boot, and Run Levels

## Boot sequence (four stages)

1. **BIOS POST** — Power On Self Test; checks hardware. If it fails, boot stops.
2. **Boot loader** — Loads from first sector of boot device. In Linux often in **`/boot`**. **GRUB2** is common; shows boot menu, loads chosen kernel into memory, passes control to kernel.
3. **Kernel initialization** — Kernel decompresses, initializes hardware and memory, then starts the **init process**.
4. **Init process** — Sets up user space. On most current distros this is **systemd** (mounts filesystems, starts and manages services). Older systems used **SysV init** (e.g. RHEL/CentOS 6). Check with: **`ls -l /sbin/init`** (systemd will point to `/lib/systemd/systemd`).

## Run levels (systemd targets)

- **Runlevel 5** — Graphical mode (GUI). In systemd: **`graphical.target`**.
- **Runlevel 3** — Non-graphical (multiuser, no GUI). In systemd: **`multi-user.target`**.

**`runlevel`** — Show current run level.  
**`systemctl get-default`** — Default target (reads `/etc/systemd/system/default.target`).  
**`systemctl set-default multi-user.target`** — Set default to non-graphical (or `graphical.target` for GUI).
