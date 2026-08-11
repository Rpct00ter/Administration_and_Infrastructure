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

## 5. Change the priority of the process with PID `1234` to `10`.

```bash
sudo renice 10 -p 1234
```
>>**NOTE**: <br />
>>-20  --> highest priority<br />
>>  0  -----> default<br />
>>+19  --> lowest priority
## 7. Terminate the 'sleep' process.

```bash
kill -1 <PID>
```

## 6. Display all open files used by the process with PID `5678`.

```bash
sudo lsof -p 5678
```

## 7. Limit the CPU usage of the specified process.

```bash
# Place for future command
```

## 8. Run a command with a time limit of **60 seconds**.

```bash
timeout 60s sleep 300
```

## 9. Check the resource limits applicable to the current shell.

```bash
ulimit -a
```

## 10. Change the soft limit for the number of open files for the current session.

```bash
ulimit -n 512
```

## 11. Display the process tree starting from the `systemd` process.

```bash
pstree -p 1
```
