# 6. Networking

*Merged: Networking, ports. See **12-concepts-explained.md** for sk_buff, RPC (NFS).*

---

## Basics

- **Interface** — eth0, lo. **IP**, **MAC**, **port** (0–65535).
- **Commands:** `ip link`, `ip addr`.

---

## OSI model (7 layers, guide)

1. **Application** — HTTP, FTP, SMTP, Telnet.
2. **Presentation** — SSL/TLS, encoding.
3. **Session** — NetBIOS, RPC.
4. **Transport** — TCP, UDP.
5. **Network** — IP, ICMP, OSPF.
6. **Data Link** — Ethernet, PPP, Wi-Fi (MAC, framing).
7. **Physical** — cables, hubs, bit stream.

---

## TCP/IP model (4 layers, guide)

1. **Application** (OSI 7–5) — HTTP, FTP, SMTP, DNS, Telnet.
2. **Transport** — TCP, UDP.
3. **Network/Internet** — IP, ICMP, ARP.
4. **Data Link/Network Access** — Ethernet, Wi-Fi.

---

## TCP, UDP, IP, ICMP (guide)

**TCP:**
- Connection-oriented, reliable, stream.
- **3-way handshake:** Client SYN → Server SYN-ACK → Client ACK.
- **Data:** segments, sequence numbers, ACKs.
- **4-way close:** A FIN → B ACK → B FIN → A ACK.

**UDP:**
- Connectionless, unreliable, datagram.
- No handshake/ACK/retransmit.
- Used for: DNS, VoIP, gaming.

**IP:** Logical addressing, routing; connectionless, best-effort.

**ICMP:** Diagnostics, error reporting; no TCP/UDP, no ports; **ping**, **traceroute**.

---

## DNS (guide: resolution process)

1. **Local cache** — app/OS, /etc/hosts.
2. **Resolver** — query to configured resolver (e.g. 8.8.8.8).
3. **Resolver cache**.
4. **Recursive:** resolver → **root** → **TLD** (.com etc.) → **authoritative** → IP (A/AAAA).
5. Resolver caches (TTL), returns to client.
6. Client uses IP.

- **UDP port 53** (TCP for large/zone transfer).
- **dig**, **nslookup**.

---

## Routing table (guide)

**Commands:** `route -n` or `ip route show`.

**Fields:**
- **Destination** — 0.0.0.0 = default.
- **Gateway** — next hop; 0.0.0.0 = direct.
- **Genmask** — netmask; 255.255.255.255 = host route.
- **Flags** — U=Up, G=Gateway, H=Host.
- **Metric** — cost; lower better.
- **Iface** — e.g. eth0, wlan0.

---

## Network commands (guide)

- **ping** — ICMP Echo Request/Reply; `ping <host>`; `-c <count>`, `-s <size>`; packet loss, RTT.
- **telnet <host> <port>** — TCP port check; if fails, port not listening or firewall.
- **traceroute <host>** — path and delay; uses increasing TTL.
- **netstat -tulnp** (listen + programs), **netstat -rn** (routing), **netstat -i** (interfaces).
- **ss -tulnp**, **ss -s** (summary); faster than netstat.
- **ip:** `ip addr show`, `ip link show`, `ip route show`.
- **nmap** — scan hosts, services, OS.
- **tcpdump** / **wireshark** — packet capture.

---

## Network card packet reception (guide)

1. NIC receives packets.
2. Hardware interrupt to CPU.
3. Kernel runs NIC **interrupt handler (top half)**.
4. Handler acks hardware, copies packets into memory (e.g. **sk_buff**), frees NIC buffer.
5. Delay here → buffer overrun → **dropped packets**.
6. Rest (IP, TCP/UDP) in **bottom half** (softirq).

(See **12-concepts-explained.md** for sk_buff.)

---

## Workflow: typing a URL in the browser (guide)

1. **URL parsing** — protocol, domain, path.
2. **DNS** — local cache → resolver → root → TLD → authoritative → IP.
3. **TCP** — connect to IP:80 (HTTP) or 443 (HTTPS); 3-way handshake; for HTTPS, TLS handshake first.
4. **HTTP(S) request** — e.g. GET /path HTTP/1.1.
5. **Server** processes request.
6. **Response** — status, headers, content.
7. **Rendering** — browser may request more resources.
8. **Teardown** — 4-way FIN/ACK.

**Packet path:** HTTP GET → TCP header → IP header (routing) → Data Link (Ethernet, MAC) → Physical.
