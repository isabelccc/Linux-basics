# 11. PE Interview Skills

Skills used every day: **text manipulation**, **I/O**, **automation**, **interfacing with systems/processes**. Map to notes 01–10. For **concept explanations** (e.g. NFS, VFS, CFS), see **12-concepts-explained.md**.

---

## Text manipulation

| Skill | Notes | Commands |
|-------|--------|----------|
| Search by pattern | 01 (find), 10 | grep, grep -E/-v/-n/-r |
| Extract columns | 01 (pipe), 10 | awk '{print $N}', awk -F:, cut |
| Substitute/transform | 10 | sed 's/old/new/', sed -i |
| Combine streams | 01 (pipe) | \|, >, >>, 2>, &>, < |
| Find files | 01 | find, find -exec, locate |

**Use:** Log analysis (grep/awk), config edits (sed), parsing ps/df/ss (awk, cut).

---

## Handling I/O

| Skill | Notes | Commands |
|-------|--------|----------|
| stdin/stdout/stderr | 01, 02 (FD) | 0,1,2; 2>&1, tee |
| Pipes and redirects | 01 | cmd1 \| cmd2, >, >>, xargs |
| File descriptors | 02, 10 | /proc/PID/fd, lsof -p PID |

**Use:** Chaining commands, logging (tee), debugging “too many open files.”

---

## Automating tasks

| Skill | Notes | Commands |
|-------|--------|----------|
| Shell scripting | 01 (bash) | #!/bin/bash, if/for/while, $? |
| One-off automation | 01 | Pipes, find -exec, xargs, watch |
| Service and boot | 05 (systemd) | systemctl enable/start, unit ExecStart |

**Use:** Scripts, systemd units/timers.

---

## Interfacing with systems

| Skill | Notes | Commands |
|-------|--------|----------|
| HTTP(S) and APIs | 06, 10 | curl, wget |
| DNS and connectivity | 06, 10 | dig, ping, traceroute, telnet/nc |
| Process control | 04, 10 | kill, systemctl restart |
| Inspecting running/listening | 04, 06, 10 | ps, top, ss -lntp, lsof -i |
| Calling programs from scripts | 01, 04 | $(cmd), PATH |

**Use:** curl health checks, dig/ping, kill/systemctl, ps/ss/lsof.

---

## Checklist

- [ ] grep, awk (fields), sed, cut, find in pipelines
- [ ] stdin/stdout/stderr, \|, >, >>, 2>&1, tee
- [ ] Bash: variables, if/for/while, $?
- [ ] systemctl, unit files
- [ ] curl, dig, ping, ss, lsof, kill
