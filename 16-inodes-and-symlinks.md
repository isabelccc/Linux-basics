# 16. Inodes and Symlinks

## Inodes

An **inode** (index node) holds a file’s **metadata** (permissions, owner, size, timestamps, pointers to data blocks), not the filename or content. The directory entry maps **filename → inode number**. Each file/directory has a unique inode number on that filesystem.

- View inode numbers: **`ls -li`** (first column).
- Detailed info: **`stat file`** (inode, links, size, etc.).
- Inode count is fixed at filesystem creation — limits number of files.

## Hard links

Multiple directory entries (filenames) pointing to the **same inode**. Same data; deleting one name does not remove data until the last link is removed. Cannot cross filesystems. Create: **`ln source linkname`**.

## Symbolic (soft) links

A separate file that stores a **path** to the target. Can point to directories and across filesystems. If target is removed, link becomes broken. Create: **`ln -s target linkname`**.

**`ls -l`** shows the target: `linkname -> target`.

## Summary

|          | Hard link     | Symlink        |
|----------|---------------|----------------|
| Inode    | Same as target| Own inode      |
| Target   | Same filesystem only | Any path   |
| Directory| No            | Yes            |
| Broken   | No            | Yes if target removed |
