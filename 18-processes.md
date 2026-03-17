# 18. Processes

## What is a process?

A **process** is a running instance of a program: code, CPU state, memory, etc. Each has a unique **PID**. The OS schedules processes (context switching); on multi-core systems each core can run processes independently.

- **Shell job** — Process started from the shell (interactive).
- **Daemon** — Background process, often started at boot, no direct user interaction (e.g. print spooler, network daemon).

## Viewing processes

- **`ps -ef`** — All processes (PID, PPID, TTY, TIME, CMD).
- **`ps aux`** — User, PID, %CPU, %MEM, VSZ, RSS, TTY, STAT, TIME, COMMAND.
- **`ps fax`** — Process tree (parent–child).
- **`top`** or **`htop`** — Real-time view (CPU, memory, sort, kill).

## Signals

- **`kill PID`** — Send TERM (15). **`kill -9 PID`** — Send KILL (9, force).
- **`killall name`** — Kill by process name.

## Background and jobs

- **`cmd &`** — Run in background.
- **`Ctrl+Z`** — Suspend foreground job. **`bg`** — Run in background. **`fg`** — Bring to foreground.
- **`jobs`** — List jobs. **`nohup cmd &`** — Run immune to hangup.
