# 22. Services and Systemd

## Services and daemons

A **service** is a background process that provides a function (e.g. SSH server, web server). A **daemon** is a long-running background process, often started at boot; many services are daemons (e.g. **sshd**, **httpd**, **crond**). They are managed by the init system (e.g. systemd).

## Systemd

**systemd** is the init and service manager on most modern Linux systems. It manages **units** (services, mounts, sockets, etc.).

- **`systemctl list-units`** — List units.
- **`systemctl start name`** — Start service. **`systemctl stop name`** — Stop.
- **`systemctl restart name`** — Restart. **`systemctl reload name`** — Reload config.
- **`systemctl enable name`** — Start at boot. **`systemctl disable name`** — Don’t start at boot.
- **`systemctl status name`** — Status and recent log lines.
- **`systemctl get-default`** — Default target (run level). **`systemctl set-default target`** — Set default.

## Unit files

Service unit files live under **/etc/systemd/system/** or **/lib/systemd/system/**. After editing: **`systemctl daemon-reload`** then **start/restart** as needed.

## Creating a systemd service

Create a unit file, e.g. **/etc/systemd/system/myservice.service**:

```ini
[Unit]
Description=My Service
After=network.target

[Service]
ExecStart=/usr/bin/myapp
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

Then: **`systemctl daemon-reload`**, **`systemctl enable myservice`**, **`systemctl start myservice`**.
