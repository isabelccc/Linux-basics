# 1. Fundamentals (Linux, Shell, Commands, User)

*Merged: Meta PE overview, Linux intro, shell, basic commands, bash, pipe/redirect, finding files, environment variables, user management.*

---

## Meta PE Interview Overview (from guide)

**What is Production Engineering at Meta?**
- PEs ensure **Reliability, Scalability, Performance, and Security** of production services.
- Hybrid software and systems engineers; bridge between infrastructure and software, teams and processes.

**Interview structure (screening):**
- Two 45-minute interviews:
  1. **Coding** — programming skills for production engineering.
  2. **PE Basics** — system knowledge.
- Format: 5 min intro, 35 min technical, 5 min Q&A.

**Assessment areas:**
- Embracing ambiguity (technical).
- **Operating systems concepts** (PE).
- **Web technology** (PE).
- Distributed systems theory.

**Key competencies:**
- Deep Linux knowledge (kernel internals beyond surface commands).
- **Troubleshooting focus** (methodical production problems).
- **Performance analysis** (tools and metrics for bottlenecks).
- **System design thinking** (how components interact at scale).
- Focus on OS fundamentals: tooling, memory management, Unix process lifecycle.

**Preparation (from guide):**
- Use Linux as primary desktop.
- Build and run simple websites on Linux.
- Work with HTTP client/server on Linux.

---

## Linux introduction

- **OS** manages: memory (MMU), processes, devices, storage, CPU (scheduling), networking.
- **Distribution** = kernel + libraries, daemons, tools (e.g. Ubuntu, Debian, Fedora).
- **Architecture:**
  - **Kernel space** — scheduling, memory, drivers, syscalls.
  - **User space** — apps, shell.
  - Shell reads commands and runs programs; access to kernel is via **system calls**.

---

## Shell

- **Shell** — Text interface; reads commands, runs programs.
- **Home directory** `~`; **prompt** shows host/dir.
- **Command types:**
  - **Built-in** (e.g. `echo`, `cd`, `pwd`).
  - **External** (e.g. `ls`, `date`).
  - **`type cmd`** to check.
- **What happens when you type `ls -l`:** Input → tokenize → alias/expand → built-in check → PATH lookup → **fork()** → child **execve()** → ls runs (open/getdents/stat, write stdout) → **exit()** → parent **wait()** → prompt.
  - Full flow for `ls -l *.c` in **10-commands-reference.md**.

---

## Basic commands

**Nav:**
- `pwd`, `ls` (-l, -a, -lt), `cd`, `mkdir` (-p), `pushd`/`popd`.

**Files:**
- `mv`, `cp` (-r), `rm` (-r), `touch`, `cat`, `more`/`less`, `head`/`tail` (-f).

**History:**
- `history`, ↑/↓, `Ctrl+R`, `!n`, `man cmd`, `apropos keyword`, Tab completion.

---

## Pipe and redirect

- **Streams:** stdin (0), stdout (1), stderr (2).
- **Redirect:** `>`, `>>`, `2>`, `<`, `2>&1`, `&>`.
- **Pipe:** `|` (stdout → next stdin); `|&` (stdout+stderr).
- **`tee`** — to file and screen.

---

## Finding files

- **find** — `find path -name "*.log"`; options: `-type f/d`, `-user`, `-size +N`, `-mtime`, `-exec`, `-delete`.
- **locate** — Fast name search (uses **updatedb**).
- **which cmd** — Path from PATH.

---

## Environment variables

- **Set/export:** `export VAR=value`; `echo $VAR`; `env` / `printenv`.
- **PATH** — Colon-separated; add dir: `export PATH=$PATH:/dir`.
- **Common:** HOME, PWD, USER, SHELL, LANG.
- **Persist:** ~/.profile or ~/.bashrc.

---

## Bash

- **Check:** `echo $SHELL`; `chsh` to change.
- Aliases, history, Tab.
- **Config:** ~/.bashrc (interactive), ~/.profile (login).

---

## User management

- **Files:** ==/etc/passwd (user, UID, GID, home, shell); /etc/shadow (passwords)==.
- **root / sudo:** `usermod -aG sudo user`; **visudo** to edit safely.
- **Switch:** `su`, `su - user`; `sudo cmd`.
- **Commands:** useradd/userdel, usermod; groupadd; **id**, **groups**.
