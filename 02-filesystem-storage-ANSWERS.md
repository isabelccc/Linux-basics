# 2. Filesystem and Storage — Answer Key

All answers keyed to **02-filesystem-and-storage.md** and **12-concepts-explained.md**. Use to check your self-test.

---

## Files and directories

**1. Absolute vs relative path; one example each.**  
- **Absolute:** Full path from root `/`. Example: `/home/user/file.txt`.  
- **Relative:** Path from current working directory. Example: `../other/file.txt` or `./local.txt`.

**2. Commands: show current dir; list files; create dir; create empty file; copy; move/rename; delete.**  
- **pwd** — current directory. **ls** — list files. **mkdir name** — create directory. **touch file** — empty file. **cp src dst** — copy. **mv src dst** — move/rename. **rm file** (or **rm -r dir**) — delete.

**3. What do `./`, `../`, and `~/` mean?**  
- **./** — current directory. **../** — parent directory. **~/** — current user’s home directory (not root).

---

## File system structure and types

**4. Difference between file and filesystem?**  
- **File** = data on disk (content of one file). **Filesystem** = the layout/organization (e.g. ext4, XFS) — how blocks, inodes, directories are structured on a partition.

**5. See type of file from CLI? List hidden files?**  
- **file path** — shows file type. **ls -a** — list all, including hidden (names starting with `.`).

**6. One sentence each: block, superblock, block groups.**  
- **Block** — fixed-size unit of storage on disk (kernel uses blocks; multiple of sector).  
- **Superblock** — on-disk structure that describes the whole filesystem (type, block size, total blocks/inodes, etc.).  
- **Block groups** — subdivision of the FS; each has superblock copy, block bitmap, inode bitmap, inode table, data blocks.

**7. Seven file types; first character in ls -ld for each.**  
- Regular (**-**), directory (**d**), symlink (**l**), character device (**c**), block device (**b**), socket (**s**), named pipe (**p**).  
- **ls -ld** first character = file type.

---

## Filesystem hierarchy (FHS)

**8. Purpose of /home, /root, /etc, /var, /dev, /proc, /sys, /bin, /usr, /lib, /tmp, /mnt, /media.**  
- **/home** — user home dirs. **/root** — root’s home. **/etc** — config. **/var** — logs, variable data. **/dev** — device files. **/proc**, **/sys** — kernel/runtime info. **/bin**, **/usr** — binaries. **/lib** — libraries. **/tmp** — temp. **/mnt**, **/media** — mount points.

**9. List all mounted filesystems with human-readable sizes.**  
- **df -h**

---

## Permissions

**10. r, w, x on file; x on directory?**  
- On **file:** r = read, w = write, x = execute. On **directory:** r = list entries, w = create/delete entries, **x = enter directory (cd)**.

**11. chmod: (a) u/g/o +/-, (b) octal 755?**  
- (a) **chmod u+x file**, **chmod g-w file**, **chmod o+r file** (u=user, g=group, o=others; + add, - remove).  
- (b) Octal: 4=read, 2=write, 1=execute. **chmod 755** = rwx for user (7), r-x for group (5), r-x for others (5).

**12. umask, chown, chgrp?**  
- **umask** — default permissions *removed* for new files/dirs (e.g. 022 → new file 644, dir 755).  
- **chown user:group file** — change owner (and optionally group). **chgrp group file** — change group only.

---

## Inodes and links

**13. What does inode store? Where does filename live?**  
- Inode stores **metadata only** (perms, owner, size, timestamps, link count, pointers to data blocks). **Filename lives in the directory** (directory = list of name → inode number), not in the inode.

**14. Hard link vs symlink: create? Cross FS? If target deleted?**  
- **Hard link:** `ln src link`. Same inode. **Cannot** cross FS. If “target” deleted, data stays (other link still valid).  
- **Symlink:** `ln -s target link`. Own inode storing path string. **Can** cross FS. If target deleted → **dangling** symlink.

**15. File descriptors 0, 1, 2? Which syscalls create/use fd?**  
- 0 = stdin, 1 = stdout, 2 = stderr. **open()** creates fd. **read()**, **write()**, **close()**, **lseek()**, **dup()** use fd.

**16. Kernel three-level view (per-process table → ? → ?).**  
- Per-process **file descriptor table** → system-wide **open file table** (file object: offset, refcount, mode) → **v-node / inode table** (file identity and metadata).

---

## Inode (detail)

**17. “Inode number unique per filesystem”? How see inode number and full metadata?**  
- Each filesystem has its own inode number space; same number can exist on different mounts. **ls -li** shows inode number. **stat path** shows full inode metadata.

**18. Six+ things inode stores.**  
- Identity (inode number, device); type & permissions (i_mode); ownership (i_uid, i_gid); size (i_size); timestamps (atime, mtime, ctime); link count; data location (block pointers / extents); FS-specific (e.g. extents, ACLs).

**19. How does kernel find file data from inode? (direct, indirect blocks)**  
- Inode has **direct block pointers** (first 12 or so) → physical blocks. For large files: **single/double/triple indirect blocks** — block that holds more block numbers. Kernel follows chain to get physical block for a given file offset.

**20. Inode on disk? In memory? Who shares in-memory inode?**  
- **On disk:** in the **inode table** inside a block group. **In memory:** kernel caches **in-memory inode** (VFS inode + FS-specific). **Shared by** every open file (file object) and every dentry that refers to this file.

**21. Why “run out of inodes” with free space? Command for inode usage?**  
- Inodes are fixed at mkfs time. Many tiny files use few blocks but one inode each → inodes exhausted while **df -h** still shows free space. **df -i** shows inode usage (total/used/free per FS).

---

## File stream and file descriptor (detail)

**22. “File stream”: (a) program’s view, (b) kernel’s view.**  
- (a) **Program:** stream of bytes to read/write (e.g. stdin, stdout, file from fopen/open).  
- (b) **Kernel:** **file descriptor** — integer indexing process fd table; each fd points to an **open file description** (file object) with offset, mode, ref to inode.

**23. Three kernel structures for fd; what each holds.**  
- (1) **Per-process fd table:** fd index → pointer to file table entry. (2) **Open file table (file object):** offset (f_pos), access mode, refcount, pointer to dentry/inode. (3) **V-node/inode table:** file identity, metadata (inode).

**24. open() vs dup(): same file table entry? Same offset?**  
- **open()** creates a **new** file table entry (new offset). **dup()** creates a **new** fd that points to the **same** file table entry → **same offset** (shared).

**25. read(fd): how kernel uses fd to get data? Where is “current position”?**  
- fd → file table entry → get **offset** (f_pos) and **inode**. Read from **page cache** at that offset (or trigger FS readpage → block layer). “Current position” lives in the **file table entry** (f_pos). After read, kernel advances f_pos.

**26. Two processes each open() same file: one or two file table entries? After fork()?**  
- **Two** file table entries (each open() gets its own) → two independent offsets. **After fork():** child gets **copy** of fd table; fds point to **same** file table entries → **shared** offset with parent.

**27. “Too many open files”? How inspect open files for a process?**  
- Process hit **per-process fd limit** (ulimit -n) or system ran out of **file table entries**. **lsof -p PID** or **ls -l /proc/PID/fd/** to list open files.

---

## Disk usage and mounting (df vs du)

**28. df reports per what? du reports per what?**  
- **df** — per **mounted filesystem** (one row per mount point: device, total, used, available, use%).  
- **du** — per **directory** (or path you give): space used by files under that path.

**29. What is mounted filesystem? Does df show space for a subdir of same mount?**  
- **Mounted filesystem** = partition/volume attached to a directory (mount point). **df** shows only mounts; for a subdir on the same mount (e.g. /var when / is the mount), **df /var** shows the **same** numbers as **df /** (same mount).

**30. df -h vs df -i.**  
- **df -h** — human-readable sizes (GiB, MiB). **df -i** — inode usage (total/used/free inodes per FS).

**31. du -sh path vs du -h --max-depth=1.**  
- **du -sh path** — total size of path (summary, human-readable). **du -h --max-depth=1** — sizes of path and its immediate subdirs (one level).

**32. Root 95% full — which command to see which directory? Would df tell you?**  
- **du -h --max-depth=1 /** (or similar) to see which directory under / uses the space. **df** only says the mount is full; it does **not** tell you the directory.

---

## Mount and fstab

**33. What does mount do? Basic syntax; type and options.**  
- **mount** attaches a device/filesystem to a directory (mount point). Syntax: **mount device dir**; **mount -t type device dir**; **mount -o options device dir**. **umount dir** (or device) to detach.

**34. /etc/fstab for? mount -a?**  
- **/etc/fstab** — list of filesystems to mount at boot (device, mount point, type, options). **mount -a** — mount all entries in fstab (except noauto).

---

## VFS — object model

**35. What is VFS and why? Four main VFS objects, one sentence each.**  
- **VFS** = kernel layer that gives one common API (open/read/write/close) for all FS types (ext4, XFS, NFS). So programs don’t care which FS a path is on.  
- **Superblock** — one per mounted FS; type, block size, mount info, FS-specific ops. **Inode** — file/dir metadata (perms, size, timestamps, block pointers). **Dentry** — directory entry (name → inode); path lookup; dcache. **File object** — per open file; offset, mode, ref to dentry and inode.

**36. dentry cache (dcache)? Negative dentries?**  
- **Dcache** — kernel cache: (parent dentry, name) → child dentry. Speeds path lookup; avoids disk for repeated lookups. **Negative dentry** — cached “name does not exist” (no inode); speeds repeated failed lookups.

**37. Relationship: File object → ? → ? → data blocks.**  
- File object → **dentry** → **inode** → data blocks.

**38. Write path from write() to disk (VFS → FS → ? → hardware).**  
- write() → sys_write() → **VFS** → **FS** (e.g. ext4_file_write) → **page cache** (mark dirty) → **block layer** (submit_bio) → **I/O scheduler** → **device driver** → hardware.

---

## VFS — how it works

**39. Path resolution for /home/user/file.txt: steps.**  
- Start at root (dentry for “/”). Walk “home” → lookup in /’s directory (dcache or FS); get dentry for home. Walk “user” in home’s directory; get dentry for user. Walk “file.txt” in user’s directory; get dentry (and inode) for file.txt. Check execute on each dir in path, read/write on final inode.

**40. Path component is symlink during resolution?**  
- VFS reads symlink’s target path from inode; continues resolution from that path (with loop detection). So symlink is followed and resolution restarts at the target path.

**41. open(path): main steps in kernel.**  
- sys_open() → path resolution (dentry + inode) → permission check → allocate **file** object (f_pos=0, dentry, inode) → optional FS open() method → allocate fd, point to file, return fd.

**42. read(fd): kernel from fd to data?**  
- fd → file table → offset (f_pos) + inode. VFS calls f_op->read (e.g. generic_file_read_iter) → check **page cache**; hit → copy to user, advance f_pos; miss → FS readpage(s) → block layer → fill cache, then copy. Offset lives in file table entry.

**43. write(fd): where first? When to disk?**  
- Writes go **first to page cache** (pages marked dirty), f_pos advances. They reach disk when **flusher threads** run (memory pressure, age) or when **sync()** / **fsync(fd)** is called (FS writepages → block layer).

---

## ext4 structure

**44. Main parts of ext4 (superblock, block groups, journal).**  
- **Superblock** — FS metadata (type, block size, total blocks/inodes). **Block groups** — each has superblock copy, block bitmap, inode bitmap, inode table, data blocks. **Journal** — log for metadata (and optionally data) for consistency after crash.

**45. What is in a block group?**  
- Superblock (copy), **block bitmap**, **inode bitmap**, **inode table**, **data blocks**.

**46. Journal used for? Two ext4 features.**  
- **Journal** — record metadata (and optionally data) changes; replay on mount after crash for consistency. **Features:** extents, delayed allocation, multiblock allocation, journal checksumming, online defragmentation.

---

## Block devices and I/O stack

**47. Block device? Sector? Block (kernel)?**  
- **Block device** — storage accessed in fixed-size blocks (disks, SSDs). **Sector** — smallest unit on device (512 B or 4 KB). **Block** — kernel unit; multiple of sector, power of two, ≤ page size.

**48. Buffer cache vs page cache today? “buffers” in free?**  
- **Page cache** — caches file **pages**. **Buffer cache** — historically for block metadata; now largely unified with page cache. In **free**, “buffers” = block device metadata; “cached” = page cache (file data).

**49. Full I/O stack for write() to hardware.**  
- write() → sys_write() → VFS → FS (e.g. ext4_file_write) → **page cache** (mark dirty) → block layer (submit_bio) → **I/O scheduler** → **device driver** → hardware.

**50. Request queue?**  
- Queue of pending I/O requests for a block device. Driver takes requests from the queue and sends them to the device.

---

## I/O schedulers

**51. Four I/O schedulers; one sentence each.**  
- **NOOP** — FIFO; minimal logic; good for SSDs/virtual (no seek). **Deadline** — read/write queues with deadlines; good for DB. **CFQ** — per-process timeslices; good for desktop/mixed. **BFQ** — fairness, low latency; good for interactive/HDDs.

**52. Check scheduler for /dev/sda? Change temporarily? At boot?**  
- **cat /sys/block/sda/queue/scheduler**. Temporarily: **echo deadline > /sys/block/sda/queue/scheduler**. Boot: kernel param **elevator=deadline**.

---

## Page cache

**53. What in page cache? Read: hit vs miss?**  
- Kernel caches **file pages** in RAM. **Hit** — data in cache; copy to user (no disk). **Miss** — read from disk into cache, then copy to user.

**54. Write-through vs write-back: faster? safer after crash?**  
- **Write-back** — update cache only, flush later → **faster**; **risk** if crash before flush. **Write-through** — update cache and disk → **slower**, **safer** after crash.

**55. Flusher threads (pdflush/kworker)? When run?**  
- Kernel threads that write **dirty** page-cache pages to disk. Run when memory low, when dirty pages are old enough, or on **sync()** / **fsync()**.

**56. View page cache (Cached, Dirty, Writeback)? Force dirty to disk?**  
- **cat /proc/meminfo** (Cached, Dirty, Writeback). **sync** — flush all dirty pages. **fsync(fd)** — flush dirty pages for that file.

**57. Drop caches (testing)? echo 1 vs 2 vs 3?**  
- **echo 1 > /proc/sys/vm/drop_caches** — drop page cache. **echo 2** — drop dentries and inodes. **echo 3** — drop both (1+2). Testing only; can hurt performance.

---

## Hard link vs symbolic link

**58. Hard link: same inode? Cross FS? Point to directory?**  
- **Same inode** (same inode number). **Cannot** cross filesystems. **Cannot** point to directory (would create loops).

**59. Symlink: where target stored? Target deleted?**  
- Target path stored in the **symlink’s inode** (as a string). If target deleted, symlink becomes **dangling**.

**60. Which link type can cross FS and point to directory?**  
- **Symbolic link** (symlink). Hard link cannot do either.

---

## Disk partitioning and filesystem (steps)

**61. Steps: new partition → filesystem → mount persistently.**  
- **lsblk** (list devices). **fdisk -l device** or **parted -l** (see partitions). **fdisk device** (n new, set size, w write) or **parted**. **mkfs.ext4 partition** (e.g. mkfs.ext4 /dev/sda1). **mkdir /mnt/mydisk**. **mount /dev/sda1 /mnt/mydisk**. Add line to **/etc/fstab** for boot (device, mount point, type, options, dump, fsck order).

---

## RAID

**62. RAID 0, 1, 5, 6, 10: min disks, capacity, fault tolerance.**  
- **0:** min 2, 100%, none. **1:** min 2, 50%, 1 disk. **5:** min 3, (n-1)*size, 1 disk. **6:** min 4, (n-2)*size, 2 disks. **10:** min 4 (pairs), 50%, 1 per mirror (if not same mirror).

**63. mdadm: create RAID 1 with two devices; check status; add spare.**  
- **mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1**. Status: **cat /proc/mdstat** or **mdadm --detail /dev/md0**. Add spare: **mdadm --add /dev/md0 /dev/sdc1**.

**64. Hardware RAID? Tool?**  
- RAID done by a **controller card** (not kernel md). Tools: **MegaCLI**, **storcli** (vendor-specific).

---

## ext4 and XFS tuning

**65. Two ext4 mount options; risk of barrier=0?**  
- **noatime**, **data=writeback** (faster, less safe). **barrier=0** — disables write barriers; **risk:** data loss if power loss and no battery-backed cache.

**66. XFS: two mount options; two admin commands.**  
- Mount: **noatime**, **inode64**, **logbsize=256k**, **largeio**. Commands: **xfs_info**, **xfs_growfs**, **xfs_fsr** (defrag).

---

## Monitoring

**67. Commands to monitor disk I/O (per device and per process)?**  
- Per device: **iostat -x 1**. Per process: **iotop** (and **pidstat -d 1**).

**68. %util in iostat? Value for saturation? await?**  
- **%util** — fraction of time device was busy. **>80%** often means saturation. **await** — average time (ms) for I/O request (latency).

---

## Short-answer mix (concepts)

**69. Why filename not in inode? Rename: inode change?**  
- So one file can have multiple names (hard links) and renaming is cheap (change directory entry only). **Rename** = change directory content (name → inode mapping); **inode number does not change**.

**70. lseek(fd, offset, SEEK_SET): which kernel structure modified?**  
- The **file table entry** (open file object): its **f_pos** (current offset) is set to the new value.

**71. dcache used for in path resolution? Negative dentry?**  
- **Dcache** — cache (parent dentry, name) → child dentry so path lookup can avoid disk. **Negative dentry** — cached “name not found” (no inode) to speed repeated failed lookups.

**72. generic_file_read_iter and readpage(s): which layer does page cache? Which talks to block layer?**  
- **VFS** (generic_file_read_iter) uses the **page cache** (check hit, copy; on miss call FS). **FS layer** (readpage/readpages) talks to **block layer** (submit_bio) to fill pages from disk.

**73. df shows 95% full for /. What next to find which directory?**  
- **du -h --max-depth=1 /** (or du -sh /*) to see which directory under / uses the space.

**74. “Too many open files”: two possible limits? How list open files?**  
- (1) **Per-process fd limit** (ulimit -n). (2) **System** ran out of file table entries. **lsof -p PID** or **ls -l /proc/PID/fd/** to list that process’s open files.

---

*Use with **02-filesystem-storage-QUESTIONS.md** and **02-filesystem-and-storage.md**.*
