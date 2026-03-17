# 7. File System Structure

## Files and filesystems

A **file** is data on disk (text, binary, etc.). A **filesystem** is how the OS organizes files on a disk/partition (e.g. ext4, XFS, Btrfs).

## File types (by purpose)

- **Ordinary files** — Text, data, code; cannot contain other files.
- **Directories** — Organize files; root is `/`; users often use `/home/username/`.
- **Device files** — Represent hardware (block vs character).
- **Links** — Hard links (same inode) and symbolic/soft links (pointer to path).

**`file path`** — Identifies type (e.g. `ASCII text`, `ELF executable`, `directory`, `symbolic link`).

## By visibility

- **Visible** — Shown by `ls`.
- **Hidden** — Name starts with `.`; use **`ls -a`**.

Filenames are **case-sensitive**. Type is often determined by content, not extension.

## Blocks and layout

- **Block** — Smallest on-disk I/O unit (e.g. 4 KiB). Filesystem uses blocks for data and metadata.
- **Superblock** — Global filesystem metadata (size, structure).
- **Block groups** — Inode bitmap, block bitmap, inode table, data blocks.
