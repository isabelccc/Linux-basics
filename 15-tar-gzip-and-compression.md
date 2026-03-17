# 15. Tar, Gzip, and Compression

## tar (archive)

**tar** bundles files and directories into one archive (preserves permissions, ownership, timestamps).

Common options:

- **`-c`** — Create archive.
- **`-x`** — Extract.
- **`-t`** — List contents.
- **`-v`** — Verbose.
- **`-f file`** — Archive filename.
- **`-z`** — gzip. **`-j`** — bzip2.
- **`-C dir`** — Change to dir before extract/create.

Examples:

```bash
tar -cvf archive.tar dir1 dir2 file.txt    # Create
tar -xvf archive.tar                       # Extract
tar -tvf archive.tar                       # List
tar -xzf archive.tar.gz -C /opt            # Extract gzipped to /opt
```

## gzip

- **`gzip file`** — Compress (replaces with file.gz).
- **`gunzip file.gz`** or **`gzip -d file.gz`** — Decompress.

Often used with tar: **`tar -czvf archive.tar.gz dir`** (create gzipped), **`tar -xzvf archive.tar.gz`** (extract).

## Common patterns

- **.tar** — tar only.
- **.tar.gz** / **.tgz** — tar + gzip.
- **.tar.bz2** — tar + bzip2 (use **`-j`** with tar).
