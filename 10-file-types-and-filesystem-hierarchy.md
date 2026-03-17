# 10. File Types and Filesystem Hierarchy

## File types

In Linux, **everything is a file** (including directories).

- **Regular file** — Data, text, binaries.
- **Directory** — Special file that contains other files.
- **Special files**:
  - **Character** — Devices like keyboard, mouse (`/dev`).
  - **Block** — Block devices like disks, RAM (`/dev`).
  - **Links** — Hard link (same data, same inode) and soft/symbolic link (path pointer).
  - **Sockets** — IPC between processes.
  - **Named pipes** — Connect one process output to another’s input.

Identify type: **`file path`** or **`ls -ld path`** (first character: `-` file, `d` dir, `l` link, `c` char, `b` block, `s` socket, `p` pipe).

## Filesystem hierarchy (FHS)

Single root `/`; tree structure:

| Path    | Purpose |
|---------|--------|
| **/home** | Home directories for users (except root). |
| **/root** | Root user’s home. |
| **/opt**  | Third-party applications. |
| **/mnt**  | Temporary mount point (often empty). |
| **/tmp**  | Temporary data. |
| **/media**| External media mount point. |
| **/dev**  | Block and character device files. |
| **/bin**  | Essential user binaries (e.g. cp, mv, mkdir). |
| **/etc**  | Configuration files. |
| **/lib, /lib64** | Shared libraries. |
| **/usr**  | User-land programs and data. |
| **/var**  | Variable data (logs, mail, etc.). |

List mounted filesystems: **`df -h`** or **`df -hP`**.
