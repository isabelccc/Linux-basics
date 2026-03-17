# 33. Storage, LVM, and Disk I/O

## Block devices and partitions

- **Block devices** — Disks and partitions (e.g. **/dev/sda**, **/dev/sdb1**). **`lsblk`** lists them in a tree. **`ls -l /dev/ | grep "^b"`** lists block devices.
- **Partition table** — MBR or GPT. **`fdisk -l /dev/sda`** — List partitions. **gdisk** is used for GPT.
- **Partition types** — Primary (bootable), Extended (container), Logical (inside extended).

## Filesystems

Common Linux filesystems: **ext4**, **XFS**, **Btrfs**. Create with **mkfs** (e.g. **mkfs.ext4 /dev/sdb1**). Mount with **mount**; add to **/etc/fstab** for boot.

## DAS, NAS, SAN

- **DAS** — Direct-attached storage (local disk).
- **NAS** — Network Attached Storage (file-level over network, e.g. NFS, SMB).
- **SAN** — Storage Area Network (block-level over network).

## LVM (Logical Volume Management)

LVM adds a layer: **PV** (physical volume) → **VG** (volume group) → **LV** (logical volume). Lets you resize and combine disks.

- **pvcreate**, **vgcreate**, **lvcreate** — Create PV, VG, LV.
- **lvextend**, **resize2fs** (or **xfs_growfs**) — Grow LV and filesystem.
- **vgs**, **lvs**, **pvs** — List VGs, LVs, PVs.

## Disk I/O

- **iostat** — CPU and disk I/O stats. **iotop** — Per-process I/O (if available).
- High I/O wait in **top** or **vmstat** indicates disk bottleneck.
