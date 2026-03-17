# 10. Commands Reference

*Merged: Commands and paths reference, essential commands, debugging, flows. See **12-concepts-explained.md** for concepts.*

---

## Essential Linux commands (Meta PE guide — complete list)

**File/nav**
- ls (-l, -a, -R, -lh), cd, pwd, mkdir, rmdir, rm (-r), cp (-r), mv, find, grep, cat, less/more, head/tail (-f), touch, chmod, chown, df (-h, -i), du (-sh).

**Processes**
- ps aux, ps -ef, ps -eo pid,ppid,stat,comm; kill (SIGTERM), kill -9, kill -HUP, kill -l; pstree (-p -a); bg/fg/jobs; nohup cmd &.

**Limits**
- ulimit -a, ulimit -u; /etc/security/limits.conf.

**System**
- uname -a, uptime, dmesg.

**Monitoring**
- vmstat, iostat, top/htop, sar, free, pidstat, mpstat.

**Network**
- ip addr, ip route; ping, telnet host port, traceroute; netstat -tulnp, ss -tulnp, ss -s; dig, nslookup; lsof -i :port; tcpdump.

**User**
- who, w, last, id, su/sudo, useradd/usermod/userdel, groupadd.

**Shell**
- | > >> < 2> &>; $(cmd); VAR=value; $$; * ? [].

---

## What happens when you type `ls -l *.c` and press Enter (guide: 11 steps)

1. **Input** — Shell reads line (e.g. getline()).
2. **Tokenization** — tokens: ls, -l, *.c.
3. **Expansion & alias** — * expanded to filenames ending in .c (opendir, readdir, closedir); alias for ls if set.
4. **Built-in check** — if ls is built-in, run directly.
5. **PATH lookup** — search PATH (colon-separated), form path (e.g. /bin/ls, /usr/bin/ls).
6. **fork() and execve()** — Shell fork(); child execve(/path/ls, [args]); execve() replaces child image with ls.
7. **Parent waits** — wait()/waitpid().
8. **ls runs** — reads filesystem (inodes, directory) and formats output.
9. **Output** — write(1, ...) to stdout.
10. **exit()** — Child exits.
11. **Shell resumes** — Parent returns from wait(), prompt shown.

---

## Ctrl+C (SIGINT) — full flow (guide)

1. User presses Ctrl+C; keyboard sends scan code.
2. Keyboard driver gets scan code, interprets as Ctrl+C.
3. **TTY driver** (foreground process group) treats it as **VINTR** (interrupt character).
4. TTY sends **SIGINT (2)** to all processes in **foreground process group**.
5. **Kernel:** Signal marked **pending** in process PCB; if process in kernel mode (e.g. syscall), handling deferred until return to user mode.
6. When process runs (or returns to user mode), kernel checks pending signals. **Disposition:** custom handler → invoked; **SIG_IGN** → ignore; **SIG_DFL** (default) → **terminate**.
7. Process terminates (or continues if handler returns). Hardware interrupt was handled quickly by ISR (user↔kernel mode switch).

---

## File descriptors — kernel representation (guide)

**FD**
- Small non-negative integer; handle for open file, socket, pipe. Per-process table.
- **Standard:** 0 = stdin, 1 = stdout, 2 = stderr.
- **open()** returns FD; **read/write/close()** use it.
- **View:** **ls -l /proc/&lt;PID&gt;/fd/**, **lsof -p &lt;PID&gt;**.

**Kernel**
1. **Descriptor table (per process)** — indexed by FD; points to **file table** entry.
2. **File table (system-wide)** — offset, status flags, **reference count**, pointer to **v-node**.
3. **V-node / inode table (system-wide)** — file metadata (stat).

---

## ELF and debugging (guide)

**ELF**
- Format for executables, shared libs, core dumps.
- **Header** (magic, class, type, machine, entry). **Program headers** (segments; load). **Section headers** (sections; link).
- **Loading:** map PT_LOAD at Vaddr; dynamic linker if needed; set stack; jump to entry.
- **Static vs dynamic** — see **09-advanced-topics.md**. **Linker** — combines object files; **loader** — OS loads into memory.

**strace**
- Trace syscalls and signals.
- **strace ./cmd**, **strace -p &lt;PID&gt;** (attach), **strace -f** (follow fork), **strace -e trace=file** or **trace=network**, **-o &lt;file&gt;**; output: name, args, return.
- **ltrace** — Library calls; **ltrace -p &lt;PID&gt;**.
- **gdb** — Uses **ptrace()**: tracer (GDB) observes/controls tracee; task_struct link.
  - **New process:** child **ptrace(PTRACE_TRACEME)** then **exec()**.
  - **Attach:** **ptrace(PTRACE_ATTACH, PID)**; GDB sends SIGSTOP.
  - **Restrictions:** cannot attach to self, multiple times, or init.
  - **Breakpoints:** GDB overwrites instruction with **INT 3 (0xCC)**; on hit, kernel sends SIGTRAP to GDB; GDB restores instruction, steps, re-inserts INT 3.
  - **next/step** — single-step or temp breakpoints.

---

## Key commands (columns)

- **top:** load average, us/sy/id/wa, PID %CPU %MEM RES S COMMAND.
- **vmstat 1:** r (runnable), b (blocked), free, si/so, bi/bo, us/sy/id/wa.
- **free -h:** total, used, free, buff/cache, available, swap.
- **iostat -x 1:** r/s, w/s, rkB/s, wkB/s, await, %util.
- **df -h**, **du -sh**. **ss -lntp**. **journalctl -u unit -f**. **dmesg**.

---

## Important paths

- **/proc** — meminfo, cpuinfo, loadavg, PID/
- **/sys** — block, bus, kernel
- **/dev**
- **/etc**
- **/var**
- **/boot** — vmlinuz, initramfs, grub
