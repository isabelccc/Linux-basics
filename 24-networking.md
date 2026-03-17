# 24. Networking

## Basics

- **Network interface** — Connection between OS and network (e.g. **eth0**, **lo**). **lo** = loopback (127.0.0.1), for local traffic.
- **IP address** — Identifies a host on the network (e.g. 192.168.1.10). IPv4: four octets; IPv6: hex groups.
- **MAC address** — Hardware address of the interface (e.g. aa:bb:cc:dd:ee:ff). View: **`ip link show`**.
- **Port** — Number (0–65535) identifying a service on a host (e.g. 22 SSH, 80 HTTP).

## Commands

- **`ip addr`** or **`ip a`** — IP addresses per interface.
- **`ip link`** — Interfaces and MAC.
- **`ping host`** — Test reachability (ICMP).
- **`ss -tuln`** or **`netstat -tuln`** — Listening TCP/UDP ports.
- **`traceroute host`** — Path to host.
- **`dig domain`** or **`nslookup domain`** — DNS lookup.
- **`hostname`** — Current hostname. **`hostnamectl`** — Set/view hostname (systemd).

## DNS

DNS resolves names to IPs. Config in **/etc/resolv.conf** (nameservers). **/etc/hosts** for local overrides.

## Troubleshooting

Check: interface up and has IP (**ip addr**), default route (**ip route**), DNS (**dig**), firewall, and service listening on the expected port (**ss**, **netstat**).
