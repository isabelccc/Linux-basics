# 34. Utilities and Encryption

## Common utilities

- **cut** — Extract columns (e.g. **cut -d: -f1 /etc/passwd**).
- **sort** — Sort lines. **uniq** — Remove adjacent duplicates (often after **sort**).
- **wc** — Line/word/byte count (**-l**, **-w**, **-c**).
- **tr** — Translate or delete characters (e.g. **tr 'a-z' 'A-Z'**).
- **head** / **tail** — First/last lines. **tail -f** — Follow file.
- **xargs** — Build and run commands from stdin (e.g. **find ... | xargs rm**).
- **tee** — Split output to file and stdout (e.g. **cmd | tee log.txt**).

## Encryption basics

- **LUKS** — Disk encryption (e.g. **cryptsetup**). Encrypts whole block devices.
- **GPG** — File and message encryption/signing (**gpg --encrypt**, **gpg --sign**).
- **SSL/TLS** — Used for encrypted connections (HTTPS, SSH). Certificates and keys are stored under **/etc/ssl/** or application config.
