# 28. Performance Monitoring

## Metrics

- **CPU usage** — How much processor is used; high usage can mean slow or stuck processes.
- **RAM** — Used vs free; low free memory and heavy **swap** use can slow the system.
- **Disk I/O** — Read/write load; high I/O wait can slow everything.
- **Load average** — Average number of runnable/waiting processes (1, 5, 15 min); compare to CPU count.

## top / htop

- **`top`** — Live view: summary (load, CPU, memory, swap) and process list (default sort: CPU). **Shift+M** sort by memory. **`top -p PID`** — One process.
- **`htop`** — Similar, often easier (scroll, filter, tree).

## Other tools

- **`vmstat 1`** — Summary: procs, memory, swap, io, system, CPU.
- **`iostat 1`** — CPU and disk I/O per device.
- **`free -h`** — Memory and swap.
- **`df -h`** — Disk space. **`du`** — Directory usage.

Use these to find CPU-heavy processes, memory growth, and I/O bottlenecks.
