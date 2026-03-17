# 30. Grep

**grep** (global regular expression print) searches input for lines matching a pattern (string or regex). Used for logs, configs, and scripts.

## Syntax

```bash
grep [OPTIONS] PATTERN [FILE...]
```

If no file is given, grep reads stdin. Pattern can be a fixed string or a regular expression.

## Common options

- **`-i`** — Ignore case.
- **`-r`** or **`-R`** — Recursive (directories).
- **`-n`** — Print line numbers.
- **`-v`** — Invert: print non-matching lines.
- **`-c`** — Count matching lines.
- **`-l`** — List only filenames with matches.
- **`-E`** — Extended regex. **`-F`** — Fixed string (no regex).
- **`-A n`** — n lines after match. **`-B n`** — n lines before. **`-C n`** — n lines before and after.

## Examples

```bash
grep 'error' /var/log/syslog
grep -i "warning" file.txt
grep -rn "TODO" src/
grep -v "^#" /etc/config
```
