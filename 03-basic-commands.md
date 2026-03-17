# 3. Basic Commands

## Navigation and directories

- **`pwd`** — Print present working directory.
- **`ls`** — List directory contents.  
  - `ls -l` — Long list.  
  - `ls -la` / `ls -a` — Include hidden files.  
  - `ls -lt` — By modification time (newest first).  
  - `ls -ltr` — By time, oldest first.
- **`mkdir dir`** — Create directory.  
  - `mkdir -p a/b/c` — Create path recursively.
- **`cd path`** — Change directory.  
  - `cd` or `cd ~` — Go to home.  
  - `cd ..` — One level up.
- **Absolute path** — From root `/` (e.g. `/home/michael`).
- **Relative path** — From current directory (e.g. `Asia/India`).
- **`pushd path`** / **`popd`** — Save/restore directory stack; alternative to repeated `cd`.

## Files

- **`mv src dest`** — Move or rename file/directory.
- **`cp src dest`** — Copy file. **`cp -r src dest`** — Copy directory recursively.
- **`rm file`** — Remove file. **`rm -r dir`** — Remove directory (use with care).
- **`touch path`** — Create empty file or update timestamp.
- **`cat path`** — Print file contents. **`cat > path`** — Write (then type; end with Ctrl+D).
- **`more path`** / **`less path`** — View file with scrolling; `less` is preferred for large files.

## Command history and shortcuts

- **`history`** — List previous commands. **`history 20`** — Last 20.
- **↑ / ↓** — Step through previous commands.
- **`Ctrl+R`** — Reverse search in history; type a substring.
- **`!n`** — Run command number `n` from history. **`!str`** — Run last command starting with `str`.
- **`^old^new`** — Re-run last command with `old` replaced by `new`.
- Leading space (with `HISTCONTROL=ignorespace`) — Do not save command in history.
- **Tab** — Auto-complete command or path. **Tab Tab** — List possibilities.
- **`man cmd`** — Manual for `cmd`. Navigate with Space/b, search with `/term`, quit with `q`.
- **`apropos keyword`** — Search manual descriptions by keyword.
