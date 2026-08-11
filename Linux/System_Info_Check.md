# System Information
## Used commands

* `free`
* `df`
* `uptime`
* `uname`
* `lsmod`
* `modinfo`
* `modprobe`

# Tasks

## 1. Check the amount of free and total RAM in the system.

```bash
free -h
```

## 2. Check how long the system has been running since the last boot.

```bash
uptime
```

## 3. Display information about the currently running kernel.

```bash
uname -a
```

## 4. Display all currently loaded kernel modules.

```bash
lsmod
```

## 5. Load the `dummy` kernel module.

```bash
sudo modprobe dummy
```

## 6. Display detailed information about the `xfs` kernel module.

```bash
modinfo xfs
```
