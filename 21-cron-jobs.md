# 21. Cron Jobs

## How cron works

The **cron daemon** (**crond**) runs in the background and every minute checks **crontab** files. When the current time matches a schedule, it runs the associated command or script.

## Crontab types

- **System** — **`/etc/crontab`** (root; format includes user field).
- **User** — **`crontab -e`** (edit), **`crontab -l`** (list), **`crontab -r`** (remove). Stored under **/var/spool/cron/crontabs/** (or similar).

## Crontab syntax (user crontab)

Five time fields + command:

```
minute (0-59)  hour (0-23)  day-of-month (1-31)  month (1-12)  day-of-week (0-7)  command
```

Examples: **`*`** (every), **`*/5`** (every 5), **`1-5`** (range), **`0 2 * * *`** (daily at 02:00).

System **/etc/crontab** uses a sixth field for the user: **`minute hour dom month dow user command`**.

Example (system): **`0 * * * * root /usr/bin/apt update`** — hourly as root.

## At (one-shot)

**`at time`** — Run a command once at a given time. **`atq`** — List jobs. **`atrm job_id`** — Remove.
