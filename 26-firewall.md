# 26. Firewall

A **firewall** controls incoming and outgoing traffic by rules (allow/drop/reject by port, IP, etc.). Common tools: **iptables** (kernel rules), **firewalld**, **ufw**.

## iptables

- **Chains:** INPUT (incoming), OUTPUT (outgoing), FORWARD (through the host).
- **`iptables -L`** — List rules. **`iptables -L -n -v`** — Numeric and verbose.
- **`iptables -A INPUT -p tcp --dport 80 -j ACCEPT`** — Allow incoming TCP port 80.
- **`iptables -D INPUT 2`** — Delete rule number 2 in INPUT.
- **`iptables -P INPUT DROP`** — Default policy (use with care).

Rules are lost on reboot unless saved. Save/restore: **iptables-save** / **iptables-restore**; on Debian often **/etc/iptables/rules.v4**.

## firewalld (RHEL/Fedora)

- **`firewall-cmd --list-all`** — Zones and rules.
- **`firewall-cmd --add-service=http --permanent`** — Allow HTTP; **--reload** to apply.

## ufw (Ubuntu/Debian)

- **`ufw enable`** / **`ufw disable`** — Enable/disable.
- **`ufw allow 22`** — Allow port 22. **`ufw status`** — Show status.
