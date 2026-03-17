# 2. Filesystem and Storage — Comprehensive Self-Test

Answer these on your own; then check against **02-filesystem-and-storage.md** and **12-concepts-explained.md**.

---

## Files and directories

1. What is the difference between an absolute path and a relative path? Give one example of each.

- absolute path is the full path for a file/dir, starting from the root
/user/files/path

- relative path : is a a path relative to current position 
../relative_path

2. Which commands would you use to: show current directory; list files; create a directory; create an empty file; copy a file; move/rename; delete?
ls 
mkdir name
touch file
cp file
mv file
rm file

3. What do `./`, `../`, and `~/` mean?
./ means go back to parent
../ go back to parent's parent
~/ go to root

---

## File system structure and types

4. What is the difference between a **file** and a **filesystem**?


file : data on disk
filesystem : A filesystem is the way the operating system organizes and stores files on disk.




5. How do you see the type of a file from the command line? How do you list hidden files?


file filename
ls -a



6. In one sentence each: what are **block**, **superblock**, and **block groups** on disk?

superblock -> inode table
block -> inode bitmap

block groups -> stores data

7. Name the seven file types (regular, directory, …). What does **ls -ld** show in the first character for each?

pipe files, socket, hard link, symbolic link, regualr files, 
diretory

it shows the type of this file

---

## Filesystem hierarchy (FHS)

8. What is the purpose of: `/home`, `/root`, `/etc`, `/var`, `/dev`, `/proc`, `/sys`, `/bin`, `/usr`, `/lib`, `/tmp`, `/mnt`, `/media`?
used to store information like temporary files, shared library, logs,
booting files, etc
keep files organized


9. How do you list all currently mounted filesystems with human-readable sizes?

df -h



---

## Permissions

10. What do **r**, **w**, and **x** mean for a file? What does **x** on a directory mean?
r -> allow read
w -> allow write

x -> allow excute


11. How do you set permissions with **chmod** using (a) u/g/o and +/-, (b) octal (e.g. 755)?
1-> read
2-> write
4-> excute

first three digits for users
second three digits for groups
then rest for others
we can  chmod + 755 to make the file read/write/excute for user
read and excute for groups and others


12. What do **umask**, **chown**, and **chgrp** do?

---

## Inodes and links

13. What does an **inode** store? Where does the **filename** live — in the inode or elsewhere?


it stores all metadata for a file (except filename)
like permisison, timestamp, users, pointer for actually position
filename lives in directory only


14. **Hard link** vs **symlink**: how is each created? Can each cross filesystems? What happens if the target is deleted?

Hard link ->no effect, cant
symlink lose the pointer , can cross


15. What are file descriptors 0, 1, and 2? Which syscalls create/use a file descriptor?

0 stdin
1 stdout
2 stderr

open creates a file descripter

16. In one line: describe the kernel’s three-level view (per-process table → ? → ?).

---

## Inode (detail)

17. What does “inode number is unique per filesystem” mean? How do you see inode number and full inode metadata?



18. List at least six things the inode stores (identity, type, ownership, size, timestamps, link count, data location, etc.).
identity, type, ownership, size, timestamps, link count, data location


19. How does the kernel find file data from an inode? (direct blocks, indirect blocks)

20. Where is the inode stored on disk? Where is it kept in memory? Who shares the in-memory inode?

21. Why can “running out of inodes” happen even when **df -h** shows free space? Which command shows inode usage?
df -i

---

## File stream and file descriptor (detail)

22. What is a “file stream” from (a) the program’s view, (b) the kernel’s view?

23. Name the three kernel data structures involved when you use a file descriptor. What does each hold (e.g. offset, refcount, inode)?
24. **open()** vs **dup()**: do they share the same file table entry? Same offset?

25. **read(fd, buf, count)**: briefly, how does the kernel use the fd to get data? Where does the “current position” live?
26. If two processes each **open()** the same file, do they share one file table entry or two? What about after **fork()**?
27. What does “too many open files” mean? How do you inspect open files for a process?

---

## Disk usage and mounting (df vs du)

28. What does **df** report (per what unit)? What does **du** report (per what unit)?
29. What is a **mounted filesystem**? Does **df** show space for a directory that is just a subdirectory of the same mount?
30. **df -h** vs **df -i**: what is the difference?
31. **du -sh path** vs **du -h --max-depth=1**: what does each show?
32. The root filesystem is 95% full. Which command do you use first to see *which directory* is using the space? Would **df** tell you that?

---

## Mount and fstab

33. What does **mount** do in one sentence? Give the basic syntax and how to specify type and options.
attach filess to mounting points
34. What is **/etc/fstab** for? What does **mount -a** do?

---

## VFS — object model

35. What is VFS and why do we need it? Name the four main VFS objects and one sentence each.
blocks
inode
dentry
file object

36. What is the **dentry cache (dcache)**? What are **negative dentries**?

37. Draw or state the relationship: File object → ? → ? → data blocks.
file object -> dentry ->inode->data blocks
38. In one line: the write path from application **write()** to disk (VFS → FS → ? → hardware).

(VFS → FS → kernel → hardware).

---

## VFS — how it works

39. **Path resolution**: to resolve `/home/user/file.txt`, what does the kernel do step by step (start at root, walk components, use dcache)?
40. What happens when a path component is a **symbolic link** during path resolution?
41. **open(path)**: list the main steps in the kernel (syscall → path resolution → ? → allocate file object → ? → return fd).
42. **read(fd)**: how does the kernel get from fd to data? (fd → file → offset + inode; then page cache / readpage; FS → block layer.)
43. **write(fd)**: where do writes go first (cache or disk)? When do they reach disk?

---

## ext4 structure

44. What are the main parts of an ext4 filesystem (superblock, block groups, journal)?
45. What is in a **block group**: what tables/bitmaps?
46. What is the **journal** used for? Name two ext4 features (e.g. extents, delayed allocation).

---

## Block devices and I/O stack

47. What is a **block device**? What is a **sector**? What is a **block** (kernel)?
48. What is the **buffer cache** today vs the **page cache**? What does “buffers” in **free** mean?
49. Write the full I/O stack for a **write()** from application to hardware (at least: VFS → FS → page cache → block layer → scheduler → driver → device).
50. What is the **request queue**?

---

## I/O schedulers

51. Name the four I/O schedulers. For each: one sentence (FIFO, deadline, fairness, best for what workload).
52. How do you check the current scheduler for **/dev/sda**? How do you change it temporarily? How at boot (kernel param)?

---

## Page cache

53. What does the kernel cache in the **page cache**? On read: cache hit vs miss — what happens?
54. **Write-through** vs **write-back**: which is faster? which is safer after a crash?
55. What are **flusher threads** (pdflush/kworker)? When do they run?
56. How do you view page cache usage (Cached, Dirty, Writeback)? How do you force all dirty pages to disk? (sync vs fsync)
57. How do you drop caches (for testing)? What does echo 1 vs 2 vs 3 do?

---

## Hard link vs symbolic link

58. Hard link: same inode or different? Can it cross filesystems? Can it point to a directory?
59. Symlink: where is the target stored? If the target is deleted, what is the symlink called?
60. Which link type can cross filesystems and point to a directory?

---

## Disk partitioning and filesystem (steps)

61. List the steps to create a new partition, put a filesystem on it, and mount it persistently (commands in order: list devices → create partition → mkfs → mount → fstab).

---

## RAID

62. Fill in: RAID 0 (min disks, capacity, fault tolerance); RAID 1; RAID 5; RAID 6; RAID 10.
63. Give the **mdadm** command to create a RAID 1 array with two devices. How do you check status? How do you add a spare?
64. What is **hardware RAID**? Name a tool used with it (e.g. MegaCLI, storcli).

---

## ext4 and XFS tuning

65. Name two useful **mount options** for ext4 (e.g. noatime, data=writeback). What is the risk of **barrier=0**?
66. For XFS: name two mount options; name two admin commands (info, grow, defrag).

---

## Monitoring

67. Which commands do you use to monitor disk I/O (per device and per process)?
68. What does **%util** mean in **iostat**? What value suggests saturation? What does **await** mean?

---

## Short-answer mix (concepts)

69. Why is the **filename not in the inode**? What happens when you rename a file (inode change or not)?
inode doesnt change
we want to have filename -> inode number
so we can obtain the rest info mation from inode metadata

70. **lseek(fd, offset, SEEK_SET)**: which kernel structure does it modify?
71. What is **dcache** used for during path resolution? What is a **negative dentry**?
72. **Generic_file_read_iter** and **readpage(s)**: which layer (VFS or FS) is responsible for the page cache? Which talks to the block layer?
73. **df** shows “95% full” for /. What do you run next to find which directory is using the space?


74. A process hits “too many open files.” What are the two possible limits (per-process and system)? How do you list its open files?


---

*End of questions. Check your answers against **02-filesystem-and-storage.md** and **12-concepts-explained.md**.*
