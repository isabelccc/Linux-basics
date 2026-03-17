# 25. Ports

## Concepts

A **port** is a number (0–65535) that identifies a service on a host. The kernel routes incoming TCP/UDP traffic to the right process based on destination port.

- **Well-known ports** — 0–1023 (e.g. 22 SSH, 80 HTTP, 443 HTTPS).
- **Registered** — 1024–49151.
- **Dynamic** — 49152–65535 (ephemeral, often used for client side).

## Listing listeners

- **`ss -tuln`** — TCP/UDP listening ports (-t TCP, -u UDP, -l listening, -n numeric).
- **`netstat -tuln`** — Same idea (legacy).
- **`lsof -i :22`** — Processes using port 22.

## Firewall

Opening or closing ports is done in the firewall (iptables, firewalld, ufw). A process can listen only if the port is free and (for external access) allowed by the firewall.
