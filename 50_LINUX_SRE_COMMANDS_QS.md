# 50 Linux Commands Questions — Medium Difficulty (SRE-Focused)

Commands and scenarios commonly used by SREs. Answer in your head or on a terminal, then check the answers at the end.

**Bold question = your answer was wrong or empty** (review these with the answers at the end).

---

## Process & system

1. How do you show all processes as a tree (parent–child) with **ps**?

```bash
     pstree
```
2. A process is hung. You want to force-kill it. Which signal do you send, and with which **kill** option?

```bash
    kill -9 PID 
    # SIGKILL
```
3. **How do you see only the 10 processes using the most memory with top (without changing default config)?**

```bash
    top          # then press Shift+M to sort by memory; or: top -o %MEM
```
4. **How do you list all listening TCP ports with ss (numeric, no DNS lookup)?**
```bash
    ss -tuln
```
5. Which **ss** option shows process name/PID using each socket?
```bash
    ss -lpn
```
6. **How do you follow (tail) the systemd journal for the nginx unit in real time?**
```bash
    journalctl -u nginx -f
```
7. **How do you show journal entries from the last 2 hours with journalctl?**
```bash
    journalctl --since "2 hours ago"
```
8. **How do you see default and active iptables rules (all chains) in one go?**
```bash
    iptables -L -n -v
```
9. **How do you list all systemd units that failed to start?**
```bash
    systemctl --failed
```
10. **How do you see which systemd target (runlevel) is the default at boot?**
```bash
    systemctl get-default
```

---

## Disk & filesystem

11. **How do you show filesystem usage in human-readable form, one line per mount, without wrapping long lines?**

```bash
    df -hP
```
12. **How do you find the 5 largest directories under /var (human-readable, one line per dir)?**
```bash
    du -h /var/* 2>/dev/null | sort -hr | head -5
```
13. **How do you see inode usage per filesystem (e.g. to check "too many files" issues)?**
```bash
    df -i
```
14. **df** shows 95% used on **/** but **du -sh /** is much smaller. What is a likely cause, and how do you confirm?

```bash
lsof | grep deleted
# possibly  Deleted file still held by process
    
```
15. **How do you list block devices in a tree (disk → partition → LVM) with a single command?**
```bash
    lsblk
```
16. **How do you see which process is keeping a given file open (e.g. /var/log/app.log)?**
```bash
    lsof /var/log/app.log
```
17. **How do you safely unmount /mnt/backup if “device is busy” (identify and stop the user)?**
```bash
    lsof /mnt/backup; fuser -km /mnt/backup   # then umount
```
18. **How do you add /dev/sdb1 to /etc/fstab so it mounts at /data at boot as ext4, without editing with a text editor (one-line append)?**
```bash
    echo '/dev/sdb1 /data ext4 defaults 0 2' | sudo tee -a /etc/fstab
```

---

## Text & logs

19. **From /var/log/syslog, how do you print only lines containing error or ERROR, with line numbers?**

```bash
    grep -n -i error /var/log/syslog
```
20. **How do you count how many lines in access.log have status code 500?**
```bash
    grep -c " 500 " access.log
```
21. **How do you replace every occurrence of foo with bar in config.txt in place (GNU sed)?**
```bash
    sed -i 's/foo/bar/g' config.txt
```
22. **With awk, how do you print the first and third column of /etc/passwd using : as the field separator?**
```bash
    awk -F: '{print $1,$3}' /etc/passwd
```
23. **How do you show the last 20 lines of app.log and keep following new lines?**
```bash
    tail -n 20 -f app.log
```
24. **How do you list only the names of files under /etc that contain the string timeout (recursive, case-insensitive)?**
```bash
    grep -ril timeout /etc
```
25. **From journalctl, how do you show only entries with priority err or higher for the last hour?**
```bash
    journalctl -p err --since "1 hour ago"
```

---

## Users, permissions & sudo

26. **How do you list all users that have a valid shell (login shell) from /etc/passwd?**
```bash
    getent passwd | awk -F: '$7~/(sh|bash)$/ {print $1}'
```
27. **How do you change the group of /opt/app to appgroup and make it group-writable (one or two commands)?**
```bash
    sudo chgrp appgroup /opt/app
    sudo chmod g+w /opt/app
```
28. **How do you run myscript.sh as user deploy (assuming you have permission)?**
```bash
    sudo -u deploy ./myscript.sh
```
29. **How do you allow user backup to run only /usr/local/bin/backup.sh as root without a password, using sudoers?**
```bash
    # In sudoers (visudo): backup ALL=(root) NOPASSWD: /usr/local/bin/backup.sh
```
30. **How do you see the numeric UID, GID, and groups for the current user in one command?**
```bash
    id
```

---

## Networking & SSH

31. **How do you see the IP address(es) of the host using ip (no extra tools)?**
```bash
    ip addr
```
32. **How do you test if port 443 on example.com is open using bash and /dev/tcp (one line)?**
```bash
    bash -c 'echo >/dev/tcp/example.com/443'
```
33. **How do you copy ~/backup.tar.gz to server.example.com as user ops into their home dir using scp?**
```bash
    scp ~/backup.tar.gz ops@server.example.com:~/
```
34. How do you use **ssh** with a specific private key file (e.g. **~/.ssh/deploy_key**) to connect to **host**?
```bash
    ssh -i ~/.ssh/deploy_key user@host
```
35. **How do you see the default route (gateway) with ip?**
```bash
    ip route show default
```
36. **How do you list only listening TCP and UDP ports with ss, with numeric addresses and no resolve?**
```bash
    ss -tuln
```

---

## Scheduling & automation

37. **What crontab line runs /opt/scripts/cleanup.sh every day at 3:30 AM?**
```bash
    30 3 * * * /opt/scripts/cleanup.sh
```
38. **How do you list the current user’s crontab without editing?**
```bash
    crontab -l
```
39. **How do you run a command once at 10 PM today using at (give the at time format)?**
```bash
    echo "/path/to/command" | at 22:00
```
40. **How do you enable a systemd service to start at boot and start it now in two commands?**
```bash
    sudo systemctl enable servicename
    sudo systemctl start servicename
```

---

## Finding & running

41. **How do you find all .conf files under /etc modified in the last 7 days?**
```bash
    find /etc -name "*.conf" -mtime -7
```
42. **How do you find and delete all .tmp files under /tmp older than 1 day (one find command)?**
```bash
    find /tmp -name "*.tmp" -mtime +1 -delete
```
43. **How do you see the full path of the python3 executable that would run if you type python3?**
```bash
    which python3
```
44. **How do you run ./script.sh so it keeps running after you disconnect (immune to SIGHUP)?**
```bash
    nohup ./script.sh &
```
45. **How do you see shared libraries required by /usr/bin/nginx?**
```bash
    ldd /usr/bin/nginx
```

---

## Performance & debugging

46. How do you show CPU and I/O statistics every 2 seconds with **iostat**?
```bash
    iostat 2
```
47. **How do you show memory and swap usage in human-readable form (one command)?**
```bash
    free -h
```
48. **How do you see which process is using the most I/O (if iotop is installed)?**
```bash
    sudo iotop
```
49. **How do you trace system calls made by pid 12345 for 10 seconds and save to trace.txt?**
```bash
    timeout 10 strace -p 12345 -o trace.txt
```
50. **How do you see the load average and how long the system has been up in one command?**
```bash
    uptime
```

---

## Answers

<details>
<summary>Click to reveal answers</summary>

1. **`ps fax`** or **`ps -e --forest`**
2. Signal **9** (SIGKILL): **`kill -9 PID`** or **`kill -KILL PID`**
3. Run **top**, then press **Shift+M** to sort by memory; or **`top -o %MEM`** (BSD-style) where supported
4. **`ss -tuln`** (t=TCP, u=UDP, l=listen, n=numeric)
5. **`ss -tulnp`** (p=process)
6. **`journalctl -u nginx -f`**
7. **`journalctl --since "2 hours ago"`**
8. **`iptables -L -n -v`** (or **`iptables-save`** for full dump)
9. **`systemctl --failed`**
10. **`systemctl get-default`**
11. **`df -hP`** (P = POSIX, no wrap)
12. **`du -h /var/* 2>/dev/null | sort -hr | head -5`** or **`du -h --max-depth=1 /var 2>/dev/null | sort -hr | head -6`**
13. **`df -i`** (inode usage)
14. Deleted file still open by a process (space not freed). Confirm: **`lsof +L1`** or **`lsof | grep deleted`**
15. **`lsblk`**
16. **`lsof /var/log/app.log`** or **`fuser -v /var/log/app.log`**
17. **`lsof /mnt/backup`** or **`fuser -vm /mnt/backup`** to find PIDs; then **kill** or **fuser -km /mnt/backup`** to kill (use with care)
18. **`echo '/dev/sdb1 /data ext4 defaults 0 2' | sudo tee -a /etc/fstab`** (then **sudo mount -a** to test)
19. **`grep -n -i error /var/log/syslog`**
20. **`grep -c " 500 " access.log`** or **`grep " 500 " access.log | wc -l`**
21. **`sed -i 's/foo/bar/g' config.txt`**
22. **`awk -F: '{print $1,$3}' /etc/passwd`**
23. **`tail -n 20 -f app.log`** or **`tail -20f app.log`**
24. **`grep -ril timeout /etc`**
25. **`journalctl -p err --since "1 hour ago"`**
26. **`awk -F: '$7!="" && $7!="/usr/sbin/nologin" && $7!="/bin/false" {print $1}' /etc/passwd`** or **`getent passwd | awk -F: '$7~/(sh|bash)$/ {print $1}'`**
27. **`sudo chgrp appgroup /opt/app`** and **`sudo chmod g+w /opt/app`** (or **`sudo chmod 775 /opt/app`** if you want rwx for owner/group)
28. **`sudo -u deploy ./myscript.sh`**
29. **`backup ALL=(root) NOPASSWD: /usr/local/bin/backup.sh`** in sudoers (via **visudo**)
30. **`id`**
31. **`ip -4 addr show`** or **`ip addr`** (or **`hostname -I`** for a short list)
32. **`bash -c 'echo >/dev/tcp/example.com/443'`** (or **`timeout 2 bash -c 'echo >/dev/tcp/example.com/443'`**)
33. **`scp ~/backup.tar.gz ops@server.example.com:~/`**
34. **`ssh -i ~/.ssh/deploy_key user@host`**
35. **`ip route show default`** or **`ip r`**
36. **`ss -tuln`**
37. **`30 3 * * * /opt/scripts/cleanup.sh`**
38. **`crontab -l`**
39. **`echo "/path/to/command" | at 22:00`** (or **`at 10pm`** then type command, Ctrl+D)
40. **`sudo systemctl enable servicename`** and **`sudo systemctl start servicename`**
41. **`find /etc -name "*.conf" -mtime -7`**
42. **`find /tmp -name "*.tmp" -mtime +1 -delete`**
43. **`which python3`** or **`type python3`**
44. **`nohup ./script.sh &`** or **`disown`** after starting, or **screen**/ **tmux**
45. **`ldd /usr/bin/nginx`**
46. **`iostat 2`**
47. **`free -h`**
48. **`iotop`** (often needs **sudo**); or **`pidstat -d 1`**
49. **`strace -p 12345 -o trace.txt`** then wait 10s and Ctrl+C; or **`timeout 10 strace -p 12345 -o trace.txt`**
50. **`uptime`**

</details>

---

*Use with **linux-prep/** notes for deeper review. Good for SRE interview prep.*
