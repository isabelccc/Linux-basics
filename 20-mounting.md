# 20. Mounting

## What is mounting?

**Mounting** makes a storage device (partition, USB, NFS share) or a pseudo-filesystem available at a **mount point** (a directory) in the tree. After mounting, you access the device’s contents through that directory.

- **Can be mounted:** Partitions, USB drives, NFS/SMB shares, pseudo-filesystems (**/proc**, **/sys**, **/dev**).
- **Boot:** Root (**/**), **/proc**, **/sys**, **/dev** are mounted at boot. **/etc/fstab** defines what to mount at startup.
- **Manual:** **`mount`** for one-off or temporary mounts.

## mount / umount

**`mount device mount_point`** — Mount device at directory. Create mount point with **`mkdir`** if needed.

- **`-t type`** — Filesystem type (e.g. ext4, ntfs, nfs). Often auto-detected.
- **`-o options`** — e.g. **ro** (read-only), **rw**, **noexec**, **uid=**.
- **`umount mount_point`** or **`umount device`** — Unmount (ensure nothing is using it).

Example: **`mount /dev/sdb1 /mnt/external`**. Then **`umount /mnt/external`**.

## /etc/fstab

Each line: device (or UUID), mount point, type, options, dump, fsck. Enables **mount point** or **mount -a** to mount at boot or on demand.
