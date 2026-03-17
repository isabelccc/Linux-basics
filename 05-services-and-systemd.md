# 5. Services and Systemd

*Merged: Services, systemd.*

---

## Services and daemons

- **Service** = background process providing a function (SSH, web).
- **Daemon** = long-running, often at boot; managed by **init** (PID 1).

---

## Systemd

**Basic usage**
- **Units:** services, mounts, sockets.
- **systemctl start/stop/restart/reload name**
- **enable/disable** (boot); **status**
- **get-default** / **set-default target**

**Unit files**
- Locations: /etc/systemd/system/, /lib/systemd/system/.
- After edit: **systemctl daemon-reload**.
- Example: [Unit] After=network; [Service] ExecStart=, Restart=on-failure; [Install] WantedBy=multi-user.target.

**Targets (guide)**
| Target | Purpose |
|--------|---------|
| emergency.target | Minimal recovery |
| rescue.target | Single-user |
| multi-user.target | No GUI |
| graphical.target | With GUI |

**Mounts**
- **/etc/fstab**; **mount -a** (except noauto).

**Boot analysis**
- **systemd-analyze** — total time
- **systemd-analyze blame** — slowest services
- **systemd-analyze critical-chain**
- **systemd-analyze plot > boot.svg**
