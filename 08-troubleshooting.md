# 8. Troubleshooting

*Merged: Scaling, production debugging scenarios. **NFS** — what it is and why we need it: see **12-concepts-explained.md**.*

---

## Scaling

- **Horizontal** — More nodes; load balancing, replication/sharding.
- **Vertical** — More CPU/RAM per node.
- **Caching** (page cache, Redis); **cache invalidation**.
- **Replication**, **sharding**.

---

## High load, low CPU

- Often **D state** (I/O).
- **vmstat 1** (r, b); **ps -eo pid,stat,comm | grep "D"**; **iostat -x 1**.
- **/proc/PID/wchan**, **lsof -p PID**; **iotop -o**.
- Causes: bad disk (dmesg, smartctl), memory/swap (vmstat si/so), NFS, interrupts.

---

## Database slow

- **Phase 1:** top, mpstat, free, pidstat -d, lsof (DB files); await, %util.
- **Phase 2:** ss (port), SHOW PROCESSLIST, InnoDB status, slow query log.
- **Phase 3:** lsof (data dirs), iotop, dd test on mount; DB logs.

---

## OOM

- **free**, **/proc/meminfo**; **dmesg | grep -i killed**; **ps aux --sort=-%mem**.
- **ulimit**, overcommit, cgroup limits.
- **/proc/buddyinfo**, **pagetypeinfo**.
- **oom_score_adj -1000** for critical; monitoring; add RAM/swap.

---

## Disk I/O slow

- **iostat -x 1**, **iotop**, **pidstat -d**.
- Queue depth, scheduler. Hot files (lsof, df -i), large files (find), fragmentation.
- **strace -p PID -e read,write**; **/proc/PID/fd**. Logs.

---

## General failures (e.g. batch on some hosts)

- Logs on executor + remote.
- Disk full (df, du), disk failed (fsck, dmesg), OOM (dmesg, free), missing dep, network (ping, telnet).
- **ls stuck** ⇒ strace ls; NFS mount ⇒ mount -l, umount.

---

## NFS troubleshooting (guide)

**What is NFS and why it matters:** See **12-concepts-explained.md**. When NFS server or network is slow/down, clients block in **D state** (e.g. high load, **ls** stuck).

**(1) Client**
- Can client reach server? **ping &lt;server&gt;**.
- If not: local name service (e.g. **nisping -u** if NIS), host info, try another client.
- Can client reach NFS on server? **rpcinfo -p &lt;server&gt;**.
- Mounted? **df -hT**, **mount | grep nfs**.

**(2) Server**
- **rpcinfo -p localhost**, **systemctl status nfs-server rpcbind**.
- **nfsd** and **mountd** running; **/etc/exports** (options, client perms); server logs.

**(3) Both**
- Firewalls; **/etc/netmasks**, **/etc/nsswitch.conf**.

**(4) Performance**
- **iostat -x** on server (long **await** on NFS disk).
- **lsof +D &lt;NFS_mount_on_server&gt;**; **pidstat -d -p &lt;PID&gt;**; **iotop** on server.
