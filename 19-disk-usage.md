# 19. Disk Usage

## df (filesystem usage)

**`df -h`** — Disk space per mounted filesystem (human-readable): total, used, available, use%, mount point.

Use it to see which filesystems are full and where they are mounted.

## du (directory/file usage)

**`du`** — Space used by files and directories.

- **`du -sh path`** — Total for path (summary, human-readable).
- **`du -h path`** — Per file/dir under path.
- **`du -x /`** — Stay on one filesystem (ignore other mounts).

Find largest directories under `/`:

```bash
du -x / 2>/dev/null | sort -nr | head -10
```

Or with human-readable sizes:

```bash
find / -maxdepth 1 -type d -exec du -sh {} \; 2>/dev/null | sort -hr
```

## Cleanup

- Remove old logs, temp files in **`/tmp`**, obsolete backups, application caches.
- Use **`find`** with **`-mtime`** to locate old files; archive or delete as needed.
