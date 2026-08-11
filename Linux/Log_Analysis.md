# Log Analysis

## Used commands
- `journalctl`
- `logger`
- `systemctl`
- `rsyslog`
- `logrotate`

# Tasks

## 1. Display the last 20 system log entries.
```bash
sudo journalctl -n 20
```

## 2. Display the last 20 log entries for the `sshd` service.
```bash
sudo journalctl -u sshd -n 20
```

## 3. Display today's authentication log entries.
```bash
sudo journalctl --since today
```

## 4. Follow system logs in real time.
```bash
sudo journalctl -f
```

## 5. Find all failed SSH login attempts.
```bash
sudo journalctl -u sshd | grep "Failed password"
```

## 6. Write a custom message named "Linux+ practice" to the system log.
```bash
logger "Linux+ practice"
```

## 7. Find all successful logins for the `root` user.
```bash
sudo journalctl | grep -E "Accepted .* for root|session opened for user root"
```

## 8. Restart the `rsyslog` service and verify if it's running.
```bash
sudo systemctl restart rsyslog
```
or if not running:
```bash
sudo systemctl enable --now rsyslog
```
then:
```bash
sudo systemctl status rsyslog
```

## 9. Display the configuration file used by `rsyslog`.
```bash
# Place for future command
```

## 10. Check the status of the `logrotate` service or configuration.
```bash
# Place for future command
```

## 11. Force log rotation.
```bash
# Place for future command
```

## 12. List all rotated log files in `/var/log`.
```bash
# Place for future command
```
## 13. Configure `auditd` to log every access to the `/opt/projects/report.txt` file.
```bash
# Place for future command
```

