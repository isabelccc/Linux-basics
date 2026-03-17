# 13. Finding Files and Patterns

## find

**`find [path...] [expression]`** — Search by name, type, size, time, user, etc.

Common options:

- **`-name pattern`** — By name (e.g. `find /var/log -name error.log`).
- **`-type f`** / **`-type d`** / **`-type l`** — File, directory, symlink.
- **`-user name`** — Owned by user. **`! -user guest`** — Not owned by guest.
- **`-size +N`** — Larger than N (blocks; 1 block = 512 bytes). **`-size -10M`** — Less than 10 MiB.
- **`-mtime -7`** — Modified in last 7 days.
- **`-exec cmd {} \;`** — Run command on each match. **`-ok`** — Prompt before running.
- **`-delete`** — Remove matches (use with care).

Examples:

```bash
find /var/log -name "*.log"
find /home -user admin
find . -type f -size +100M
```

## locate

**`locate pattern`** — Fast search by filename using a database (updated periodically with **updatedb**). Less precise than **find** but quicker for name-only search.

## which

**`which cmd`** — Path of the executable used when you run **cmd** (first match in **PATH**).
