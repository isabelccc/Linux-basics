# 7. Performance and Logs

*Merged: Performance monitoring, log files. See **12-concepts-explained.md** for NUMA.*

---

## Methodology (guide)

**USE (Brendan Gregg)** — Per resource (CPU, memory, disk, network):
- **Utilization** (how busy)
- **Saturation** (queue length, e.g. run queue for CPU, I/O queue for disk)
- **Errors** (e.g. I/O errors, drops)

**RED (Tom Wilkie)** — Per service:
- **Rate** (requests/sec)
- **Errors** (failed/sec)
- **Duration** (latency distribution)

---

## vmstat (guide: columns)

**vmstat &lt;interval&gt; [count]** (e.g. vmstat 1 5)

| Section | Column | Meaning |
|---------|--------|---------|
| **procs** | r | Runnable (if r &gt; #CPUs → CPU saturation) |
| | b | Uninterruptible sleep (high = I/O bottleneck) |
| **memory** | swpd, free, buff, cache, inact, active | KB |
| **swap** | si | Swap in/sec |
| | so | Swap out/sec; ideally 0; &gt;10 problematic |
| **io** | bi, bo | Blocks in/out |
| **system** | in | Interrupts/sec |
| | cs | Context switches/sec |
| **cpu** | us, sy, id, **wa** (I/O wait; &gt;20% notable), st | Percent |

**Interpretation:** r &gt; cores = CPU bottleneck; persistent b &gt; 0 = I/O bottleneck; si/so &gt; 0 = memory pressure.

---

## iostat (guide)

**iostat -x &lt;interval&gt; [count]**

Per device:
- **r/s, w/s** — IOPS
- **rkB/s, wkB/s**
- **rrqm/s, wrqm/s**
- **r_await, w_await** (ms); **await**
- **avgqu-sz** — queue length
- **rareq-sz, wareq-sz**
- **svctm** (deprecated)
- **%util** — &gt;80% = saturation

**Thresholds:** await &lt; 1 ms (good SSD), &lt; 10 ms (OK SSD); &lt; 20 ms (good HDD), &lt; 50 ms (OK HDD).

---

## top / htop (guide)

**Header:** Uptime, users, **load average (1, 5, 15 min)**. Tasks: total, running, sleeping, stopped, zombie.

**CPU:** us, sy, ni, id, wa, hi, si, st. Memory: total, used, free, buffers, Swap; cached.

**Process columns:** PID, USER, PR, NI, VIRT, RES, SHR, S (state), %CPU, %MEM, TIME+, COMMAND.

**Interactive:** P (CPU sort), M (MEM), T (TIME), k (kill), r (renice), f (fields), 1 (per-CPU).

**Load average** = average processes in R or D over 1/5/15 min; &lt; 1 per core = underutilized; = 1 = utilized; &gt; 1 = overloaded; **high load + low CPU** = I/O bottleneck (many D).

---

## sar, free, pidstat, mpstat, CPU tuning (guide)

**sar** (sysstat)
- **sar -u 1 10** (CPU), **sar -r 1 10** (mem/swap), **sar -b 1 10** (block I/O), **sar -q 1 10** (queue, load), **sar -n DEV 1 10** (network), **sar -P ALL 1 10** (per-CPU).
- Historical: **sar -u -f /var/log/sa/sa01**.

**free -h** — "available" in newer versions = estimate for new apps.

**pidstat**
- **pidstat -u 1 &lt;PID&gt;** (CPU), **pidstat -r 1 &lt;PID&gt;** (mem), **pidstat -d 1 &lt;PID&gt;** (I/O), **pidstat -w 1** (context switches).

**mpstat -P ALL 1**. **lscpu**, **numactl --hardware**.

**CPU frequency**
- **cpupower frequency-info**.
- **cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor**
- **echo performance &gt; .../scaling_governor**
- Governors: performance, powersave, ondemand, conservative, userspace.

**Affinity and RT**
- **taskset -c 0,1 ./cmd** (affinity); **taskset -p &lt;mask&gt; &lt;PID&gt;**.
- **chrt** for real-time (FIFO, RR).

---

## Logs

- **/var/log/** — syslog, auth.log, app logs.
- **journalctl** — -u unit, -f, -n, -b, --since.
- **tail -f**, **grep**.
- **rsyslog** — /etc/rsyslog.conf.
