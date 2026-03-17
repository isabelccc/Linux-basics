# 29. Log Files and Journals

## What are logs?

Logs are timestamped records of events (service, severity, message). Used for debugging, monitoring, and security.

## Plain text logs

Often under **/var/log/** (e.g. **syslog**, **auth.log**, **apache2/access.log**). View with **less**, **tail -f**, **grep**.

## journald (systemd)

**journald** stores logs in a binary journal. Query with **journalctl**:

- **`journalctl`** — All. **`journalctl -u unit`** — For a unit. **`journalctl -f`** — Follow.
- **`journalctl -n 50`** — Last 50. **`journalctl --since "1 hour ago"`** — Time range.
- **`journalctl -p err`** — Priority error and above.

## rsyslog

**rsyslog** can write to files under **/var/log** and forward logs. Config: **/etc/rsyslog.conf** and **/etc/rsyslog.d/**.

## Practical use

- **`tail -f /var/log/syslog`** — Follow system log.
- **`grep ERROR /var/log/app.log`** — Search for errors.
- **`journalctl -u nginx -f`** — Follow nginx service logs.
