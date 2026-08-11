# Process Management
## Used commands

* `ps`
* `top`
* `pgrep`
* `kill`
* `killall`
* `nice`
* `renice`
* `jobs`
* `bg`
* `fg`
* `lsof`
* `timeout`

# Tasks

## 1. Use the 'sleep 6000' command in the background. Find process ID (PID) and terminate it.

```bash
sleep 6000 &
pgrep -a sleep
kill <PID>
```

## 2. Display the processes consuming the most RAM.

```bash
ps aux --sort=-%mem | head
```

## 3. Display the process consuming the most CPU time.

```bash
ps aux --sort=-%cpu | head
```

## 4. Move a running process from the background to the foreground.

```bash
# Check background jobs:
jobs
```
```bash
# Sends first displayed job to the foreground
fg %1
```

## 6. Change the priority of the process with PID `1234` to `10`.

```bash
# Place for future command
```

## 7. Send the `SIGHUP` signal to the `nginx` process.

```bash
# Place for future command
```

## 8. Display all open files used by the process with PID `5678`.

```bash
# Place for future command
```

## 9. Limit the CPU usage of the specified process.

```bash
# Place for future command
```

## 10. Run a command with a time limit of **60 seconds**.

```bash
# Place for future command
```

## 11. Check the resource limits applicable to the current shell.

```bash
# Place for future command
```

## 12. Change the soft limit for the number of open files for the current session.

```bash
# Place for future command
```

## 13. Display the process tree starting from the `systemd` process.

```bash
# Place for future command
```
