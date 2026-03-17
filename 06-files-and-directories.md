# 6. Files and Directories

## Paths

- **Absolute path** — From root `/` (e.g. `/home/user/notes/file.txt`). Same from anywhere.
- **Relative path** — From current directory (e.g. `notes/file.txt`). Shorter but depends on `pwd`.

## Navigation and listing

- **`pwd`** — Print working directory.
- **`cd path`** — Change directory. **`cd ~`** or **`cd`** — Home.
- **`ls`** — List contents. **`ls -al`** — Long list including hidden (permissions, owner, size, date).
- **`ls /path/*.txt`** — List files matching a pattern.

## Creating and copying

- **`touch file`** — Create empty file.
- **`mkdir dir`** — Create directory. **`cp file dir/`** — Copy file. **`cp -r src dest`** — Copy directory recursively.

## Moving and removing

- **`mv src dest`** — Move or rename.
- **`rm file`** — Remove file. **`rm -r dir`** — Remove directory (use with care).

## Special directory names

- **`./`** — Current directory (e.g. `./script.sh`).
- **`../`** — Parent directory.
- **`~/`** — Current user's home (e.g. `~/Documents`).
