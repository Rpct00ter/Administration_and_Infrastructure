# Cron and AT Task Scheduling
## Used commands

* `crontab`
* `at`
* `atq`

# Tasks

## 1. Create a `cron` job that runs every day at **3:24 AM**.
>**Note**: Ok, first of all I created a simple script that logs the current system parameters and named it 'daily_check.sh'. Here is content of 'daily_check.sh':
```bash
#!/bin/bash

LOG="$HOME/daily_system_check.log"

echo "====================================" >> "$LOG"
echo "System check: $(date)" >> "$LOG"
echo "====================================" >> "$LOG"

echo "--- Uptime ---" >> "$LOG"
uptime >> "$LOG"

echo "--- Memory ---" >> "$LOG"
free -h >> "$LOG"

echo "--- Disk usage ---" >> "$LOG"
df -h >> "$LOG"

echo "--- CPU ---" >> "$LOG"
top -bn1 | head -n 5 >> "$LOG"
```
>**Note**: Then I made it executable:
```bash
chmod +x ~/daily_check.sh
```
User's crontab editor:
```bash
crontab -e
```
Added new line:
```bash
24 3 * * * /home/developer/daily_check.sh
```
## 2. Display all `cron` jobs scheduled by the `root` user.

```bash
# Place for future command
```

## 3. Remove all `cron` jobs belonging to the current user.

```bash
# Place for future command
```

## 4. Schedule a one-time command using `at` and display the list of pending jobs.

```bash
# Place for future command
```
