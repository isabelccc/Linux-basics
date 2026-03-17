# 2. Filesystem and Storage

*Merged: Files/dirs, FS structure, file types/hierarchy, permissions, inodes/symlinks, disk usage, mounting, storage/I/O. See **12-concepts-explained.md** for VFS, page cache, buffer cache, slab.*

---

## Files and directories

- **Paths:** Absolute (from `/`) vs relative.
- **Commands:** pwd, ls, cd, mkdir, touch, cp, mv, rm.
- **Special:** ./ , ../ , ~/

---


## File system structure and types

- **File** = data on disk; **filesystem** = layout (ext4, XFS).
- **`file path`** — shows type. **Hidden:** name starts with `.`; `ls -a`.
- **Block, superblock, block groups** — layout (e.g. ext4):

```text
Filesystem
└── Block groups
      ├── superblock (copy)
      ├── block bitmap
      ├── inode bitmap
      ├── inode table
      └── data blocks
```

- **Superblock** — describes the filesystem (type, block size, total blocks/inodes, etc.).
- **Block group** — stores actual data; each has (copy of) superblock, bitmaps, inode table, data blocks.
- **Block bitmap** — tracks which data blocks in the group are free. **Inode bitmap** — tracks which inodes are free.
- **Types:** Regular, directory, character/block device, hard link, symlink, socket, named pipe.
- **ls -ld** — first char: - d l c b s p.

---

## Filesystem hierarchy (FHS)

| Path     | Purpose        |
|----------|----------------|
| /home    | User homes     |
| /root    | Root home      |
| /etc     | Config         |
| /var     | Logs, variable data |
| /dev     | Devices        |
| /proc, /sys | Kernel info |
| /bin, /usr | Binaries     |
| /lib     | Libraries      |
| /tmp, /mnt, /media | Temp, mounts |

- **df -h** — See “Mounted filesystems” and “df vs du” below.

---

## Permissions

- **r w x** for **user, group, others**.
- On dir: x = enter.
- **chmod** — u/g/o +/- rwx or octal (e.g. 755).
- **umask**. **chown**, **chgrp**.

For directories (important interview)

Directory permissions mean different things.

```
bit | meaning on directory
r | list files
w | create/delete files
x |enter directory (cd)

```
---

## Inodes and links

- **Inode** — 
Metadata (perms, owner, size, timestamps, block pointers); filename in directory entry. **ls -li**, **stat**.
- **Hard link** — Same inode; `ln src link`; cannot cross FS.
- **Symlink** — Path in own inode; `ln -s target link`; can break.
- **File descriptor (fd)** — Integer handle the kernel gives your process for an open file, socket, or pipe. Small non-negative number; **per process** (each process has its own fd table).
- **Standard fds:** 0 = stdin, 1 = stdout, 2 = stderr (always open for a process).
- **open(path, flags, mode)** — Opens a file; returns a new fd (lowest free number). **close(fd)** — Releases the fd; kernel may free the open file entry when refcount reaches 0.
- **read(fd, buf, count)** / **write(fd, buf, count)** — I/O using that fd; kernel uses fd to find the **file** object (offset, inode) and does the actual read/write (often via page cache). **lseek(fd, offset, whence)** — Changes current position for that fd.
- **dup(fd)** / **dup2(old, new)** — Create another fd that refers to the **same** open file (same offset, shared with original until one closes).
- **Inspect:** **ls -l /proc/&lt;PID&gt;/fd/** — Shows fd number → symlink to file/socket/pipe. **lsof -p &lt;PID&gt;** — Lists open files with type, device, inode, name.
- **Kernel:** per-process descriptor table → file table (offset, refcount) → v-node/inode table. (Full detail in **File stream and file descriptor** below.)

---

## Inode (important concept — detail)

**What it is**
- **Inode** = index node: kernel structure that holds **all metadata** for one file or directory (except its name). The **name** lives in the **directory** that contains the file (directory = list of name → inode number).
- Each file/dir has a unique **inode number** per filesystem. **ls -li** shows it; **stat path** shows full inode metadata.

**What the inode stores**
- **Identity:** inode number, device (which FS).
- **Type & permissions:** i_mode (file type: regular, dir, symlink, etc. + rwx for user/group/other).
- **Ownership:** i_uid, i_gid.
- **Size:** i_size (bytes).
- **Timestamps:** i_atime (last access), i_mtime (last modify), i_ctime (inode change).
- **Link count:** number of hard links; when 0, inode and data can be freed.
- **Data location:** pointers to blocks (or extents) where file content lives. First 12 direct block pointers; then single/double/triple indirect blocks for large files.
- **FS-specific:** e.g. ext4 extent tree, ACLs, extended attributes.

**On-disk vs in-memory**
- **On disk:** Inode is stored in the **inode table** (in a block group). FS driver reads it when the file is first accessed.
- **In memory:** Kernel keeps an **in-memory inode** (VFS inode + FS-specific data). Cached for speed; shared by every open file and dentry that refers to this file.
- **Important:** The **filename is not in the inode**. It lives only in the directory entry (dentry). So renaming = change directory content; inode number unchanged.

**Why it matters**
- One inode → one set of metadata and one set of data blocks. Multiple names (hard links) = same inode. **stat**, **ls -li**, and the kernel’s path→inode lookup all rely on inodes.
- **df -i** shows inode usage; running out of inodes (many tiny files) can prevent new files even if disk space remains.

---

## File stream and file descriptor (important concept — detail)

**What a “file stream” is**
- From the program’s view: a **stream** of bytes you read from or write to (stdin, stdout, a file opened with **fopen()**/ **open()**). The C **FILE*** (stdio) wraps a **file descriptor** and adds buffering.
- At the kernel level: the “stream” is represented by a **file descriptor (fd)** — an integer that indexes into the process’s **file descriptor table**. Each fd refers to one **open file description** (kernel **file** object).

**Kernel data structures (three levels)**
1. **Per-process file descriptor table**
   - Each process has a table: fd 0, 1, 2, 3, … → pointer to an **open file description** (file table entry).
   - **open()** allocates the lowest free fd and points it to a (possibly new) file table entry.
   - **dup()** / **dup2()** create a new fd that shares the same file table entry (same offset, same flags).

2. **System-wide open file table (file object)**
   - One entry per **open()** that hasn’t been **close()**d (or duplicated). Shared by fds from **dup** or **fork** (child inherits parent’s fds → same file table entry).
   - Holds: **current offset** (where the next read/write will happen — this is the “position” in the stream), **access mode** (O_RDONLY, O_WRONLY, O_APPEND, etc.), **reference count**, pointer to the **dentry** (and thus inode).

3. **V-node / inode table**
   - The actual file identity and metadata (inode). All opens of the same file share one inode; each **open()** usually creates a **new** file table entry (so each fd has its **own** offset unless shared via dup/fork).

**How read/write use it**
- **read(fd, buf, count):** Kernel uses fd → file table → get offset and inode; reads from **page cache** (or triggers disk read) at that offset; updates offset in file table; returns bytes.
- **write(fd, buf, count):** Same path; writes go to page cache (and/or disk); offset advances. So the “stream” is: same inode (file content) + per-open **offset** in the file table.
- **lseek(fd, offset, SEEK_SET)** just changes the offset in the file table entry.

**Why it matters**
- **Two processes** opening the same file get **two** file table entries → two independent offsets. **fork()** gives child a **copy** of the fd table, but each fd points to the **same** file table entry → **shared offset** (typical for pipes/sockets or coordinated use).
- **“Too many open files”** = process hit fd limit (ulimit -n) or system ran out of file table entries. **lsof -p PID** shows which inodes those fds refer to.

---

## Disk usage and mounting

**df — disk free (mounted filesystems)**
- **df** = “how much space is on each **mounted** filesystem?” It reports **per mount**: device, total size, used, available, use %, mount point.
- **Mounted filesystem** = a partition or volume that has been attached to a directory (mount point) so you can access its contents under that path. **df** only shows these; it does **not** show space for directories that are just subdirs of the same mount.
- **df -h** — Human-readable (GiB, MiB). **df -i** — Inode usage (total/used/free inodes per FS).
- Example: `df -h` might show `/dev/sda1` mounted on `/` with 50G used, 20G available. If `/var` is on the same partition, **df -h /var** shows the **same** numbers as **df -h /** (same mount).

**du — disk usage (directory trees)**
- **du** = “how much space do the **files** in this directory (and subdirs) use?” It walks the directory tree and sums file sizes. So it’s **per directory**, not per mount.
- **du -sh path** — Total for path (summary, human-readable). **du -h --max-depth=1** — One level of subdirs with sizes.
- Example: `du -sh /var/log` = total size of all files under `/var/log`. Useful for finding big dirs (e.g. which app uses space under `/var`).

**df vs du (comparison)**

| | **df** | **du** |
|---|--------|--------|
| **Stands for** | Disk free | Disk usage |
| **Reports** | Space per **mounted filesystem** (partition/volume) | Space used by **files** under a directory |
| **Unit** | One row per **mount point** | One row per **directory** (or file) you ask for |
| **Answers** | “How full is this disk/partition?” “Where can I mount something?” | “How big is this folder?” “What’s using space here?” |
| **Typical use** | Check if root (or /var, /home) is full; see all mounts | Find large dirs; cleanup; see size of a project dir |
| **-h** | Human-readable (G/M/K) | Same |
| **Ignores** | Doesn’t care about individual dirs inside a mount | Doesn’t know about mount boundaries (sums files only) |

- **Important:** If **df** says a filesystem is 95% full, use **du** (e.g. `du -h --max-depth=1 /`) to see **which directory** under that mount is using the space. **df** won’t tell you the directory; **du** won’t tell you which disk is full.

- **Cleanup:** old logs, /tmp, caches.
- **Mount** — Attach device/FS at directory. `mount device dir`; `-t type`, `-o options`; **umount**.
- **/etc/fstab** for boot; **mount -a**.

---

## VFS (Virtual File System) — object model

**What:** Kernel abstraction so open/read/write/close work the same on ext4, XFS, NFS, etc. (See **12-concepts-explained.md**.)

**Objects:**
1. **Superblock** — one per mounted FS; type, block size, mount info, total blocks/inodes, FS-specific ops.
2. **Inode** — file/dir metadata: i_mode, i_uid, i_gid, i_size, i_atime/i_mtime/i_ctime, link count, pointers to data blocks/extents; built in memory when file is accessed.
3. **Dentry** — directory entry (name → inode); path lookup. **Dentry cache (dcache):** hash + LRU.
   - **Dentry states:** Used (valid inode, active users), Unused (valid inode, no users; LRU), Negative (path doesn’t exist; speeds failed lookups).
4. **File object** — per open file; created by open(), destroyed by close(); holds offset (lseek()), access mode, ref to dentry and inode.

**Relationship:** File → Dentry → Inode → data blocks.

**Write path:** write() → sys_write() → VFS → FS write method → physical device.

---

## VFS — how it works (important concept — detail)

**Role of VFS**
- All file operations go through VFS. VFS uses **generic** logic and calls **FS-specific** functions (e.g. ext4_readpages, nfs_write) so the rest of the kernel doesn’t care whether the mount is ext4, XFS, or NFS.

**Path resolution (name → inode)**
- **Goal:** Turn a path like **/home/user/file.txt** into a **dentry** and **inode**.
- **Steps:**
  1. Start at process root (or “/”). Root has a dentry (and inode) for “/”.
  2. **Walk components:** For each component (“home”, “user”, “file.txt”):
     - Look up component in the **current dentry’s** directory (current = inode of current path so far). That means: read directory content (or use dcache); find **name → inode number**.
     - **Dcache:** Kernel keeps a hash of (parent dentry, name) → child dentry. If dentry is cached, use it (no disk). If not, ask FS to look up name in directory (FS reads directory inode’s data blocks), create dentry, put in dcache.
     - **Negative dentry:** If “name” doesn’t exist, VFS can create a **negative dentry** (no inode) and cache it so repeated lookups of missing paths are fast.
  3. After last component: you have the **dentry** for the target; dentry points to **inode**. Permissions checked against this inode (e.g. execute bit on each directory in path, read/write on final inode).
- **Symbolic links:** If a component is a symlink, VFS reads the link target (path string from inode), and continues resolution from the target path (with loop detection).

**open() flow**
1. **Syscall:** open(path, flags, mode) → **sys_open()** in kernel.
2. **Path resolution:** As above; path walk yields **dentry** (and inode). If create (O_CREAT), FS may create new inode and dentry when last component is missing.
3. **Permission check:** VFS/FS check inode (and ACLs) against flags (read/write).
4. **Allocate file object:** Kernel allocates a **file** structure; sets f_mode, f_flags, **f_pos = 0** (offset), f_dentry = dentry, f_inode = inode; links to file ops from inode’s FS.
5. **FS-specific open:** VFS calls inode’s **open()** method if defined (e.g. NFS may open on server).
6. **Install fd:** Allocate free fd in process’s fd table; point fd to this file table entry; return fd to user.

**read() / write() flow**
1. **Syscall:** read(fd, buf, count) → **sys_read()**. Kernel gets **file** from fd; gets **inode** from file→dentry→inode; gets **offset** from file→f_pos.
2. **VFS:** Calls **file→f_op→read()** (or read_iter). For a disk FS, that’s usually **generic_file_read_iter()** which:
   - Uses **page cache**: see if the needed file range is already in cache (pages). If **hit**, copy from cache to user buf and advance f_pos.
   - If **miss**, call FS’s **readpage(s)** to read from disk into page cache, then copy to user.
3. **FS layer:** ext4 (or XFS, etc.) translates file offset → logical blocks → physical blocks (using inode’s block pointers or extent tree), submits I/O via **submit_bio()** to block layer.
4. **Block layer:** I/O scheduler, driver; data comes back, fills page cache; VFS copies to user and returns.
- **write()** is similar: **file→f_op→write()** → usually **generic_file_write_iter()** → write into **page cache** (mark pages dirty), advance f_pos; flusher threads or **fsync()** push dirty pages to disk via FS **writepages()**.

**Summary**
- **VFS** = single API (open/read/write/close, etc.) and single set of in-memory objects (dentry, inode, file). **Path resolution** turns path → dentry/inode using dcache and FS lookup. **Read/write** go VFS → FS-specific readpage/writepages → block layer. So “how it works” = path walk (dentry cache + FS lookup) + file table (offset per open) + page cache + FS and block layer for real I/O.

---

## ext4 structure (guide)

- Superblock.
- **Block groups** — each: inode table, block bitmap, inode bitmap, data blocks.
- **Journal** — metadata (or full data) for consistency after crash.
- **Features:** extents, delayed allocation, multiblock allocation, journal checksumming, online defragmentation.

---

## Block devices and I/O stack

- **Block device** — data in fixed-size blocks (disks, SSDs).
- **Sector** — smallest unit on device (512 B or 4 KB).
- **Block** — kernel unit; multiple of sector, power of two, ≤ page size.
- **Buffers (buffer cache)** — blocks in memory; now largely unified with page cache; “buffers” in **free** = block device metadata.
- **Full I/O stack (write):** Application write() → sys_write() → VFS (vfs_write()) → FS (e.g. ext4_file_write()) → **page cache** (mark dirty) → block layer (submit_bio()) → **I/O scheduler** → device driver → hardware.
- **Request queue** — holds pending I/O; driver takes requests and sends to device.

---

## I/O schedulers (all four)

- **NOOP** — FIFO; minimal CPU; best for SSDs/virtual (no seek).
- **Deadline** — separate read/write queues; per-request deadline; good for DB.
- **CFQ** — per-process timeslices; good for desktop/mixed.
- **BFQ** — low latency, fairness; good for interactive/soft real-time; often default on HDDs.

**Check/change:**
- `cat /sys/block/sda/queue/scheduler`
- `echo deadline > /sys/block/sda/queue/scheduler`
- Boot: **elevator=deadline** kernel param.

---

## Page cache (guide detail)

- Kernel caches file **pages** in RAM.
- **read/write:** (1) Check page cache. (2) **Hit** → use RAM. (3) **Miss** → read/write disk, store in cache.
- **Write-through** — update cache and disk at once (slower, safer).
- **Write-back** — update cache only, mark “dirty”; flush later (faster; risk on crash).
- **Eviction:** active list (hot) vs inactive list (cold); LRU-like.
- **Flusher threads** (pdflush/kworker) — write dirty pages when memory low, dirty age threshold, or **sync()**/ **fsync()**.

**Commands:**
- **View:** `cat /proc/meminfo` (Cached, Dirty, Writeback).
- **Sync:** **sync** (all), **fsync(fd)** (one file).
- **Drop (testing only):** echo 1 = page cache, echo 2 = dentries/inodes, echo 3 = all.

---

## Hard link vs symbolic link (guide)

**Hard link:**
- Directory entry to same inode; same inode number; inode ref count.
- Data deleted when count = 0.
- **Cannot cross FS**, cannot link to dir (loops).

**Symbolic link:**
- Own inode; stores path string.
- If target deleted = dangling.
- **Can cross FS**, can point to dir.

---

## Disk partitioning and filesystem (steps)

1. **lsblk** — list block devices.
2. **fdisk -l <device>** or **parted -l** — partitions.
3. **fdisk <device>** (n new, set sectors, w write) or **parted** — create partition.
4. **mkfs.<fstype> <partition>** (e.g. mkfs.ext4 /dev/sda1).
5. **mkdir /mnt/mydisk**.
6. **mount /dev/sda1 /mnt/mydisk**; add to **/etc/fstab** for persistence.

---

## RAID (guide: levels and mdadm)

| Level | Min disks | Capacity   | Fault tolerance |
|-------|-----------|------------|-----------------|
| **0** | 2         | 100%       | None            |
| **1** | 2         | 50%        | 1 disk          |
| **5** | 3         | (n-1)*size | 1 disk          |
| **6** | 4         | (n-2)*size | 2 disks         |
| **10**| 4 (pairs) | 50%        | if not both in same mirror |

**mdadm:**
- `mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1`
- `cat /proc/mdstat` — status
- `mdadm --detail /dev/md0`
- `mdadm --add /dev/md0 /dev/sdc1` (spare)
- `mdadm --fail` / `--remove`

**Hardware RAID** — controller card; tools e.g. MegaCLI, storcli (see **12-concepts-explained.md**).

---

## ext4 and XFS tuning (guide)

**ext4:**
- Mount: **noatime**, **data=writeback** (faster, less safe), **barrier=0** (risky without battery cache), **nobh**.
- tune2fs: **journal_data_writeback**, **^has_journal** (remove journal — very risky).

**XFS:**
- Mount: **noatime**, **attr2**, **inode64**, **logbsize=256k**, **largeio**.
- **xfs_info**, **xfs_growfs**, **xfs_fsr** (defrag).

---

## Monitoring

- **iostat -x 1**, **iotop**.
- High **%util** (e.g. >80%) = saturation; **await** = latency.
