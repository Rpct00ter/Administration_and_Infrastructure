# 1. Linux Administration

> **Scope:** Debian-based Linux systems. Most commands also apply to Ubuntu and other distributions, but package managers, service names and configuration paths may differ.
>
> **Safety:** Diagnose before changing anything. Commands using `sudo`, disk tools or recursive deletion can affect the whole system. Back up important configuration before modifying it.

## 1.1 Filesystem and Essential Directories

Linux uses a single directory tree beginning at `/`. Disks and partitions are attached to this tree through mount points.

| Directory | Purpose |
|---|---|
| `/` | Root of the filesystem |
| `/etc` | System-wide configuration files |
| `/home` | Regular users' home directories |
| `/root` | Root user's home directory |
| `/var/log` | System and application logs |
| `/var` | Variable data, logs, caches and queues |
| `/tmp` | Temporary files |
| `/usr` | Installed programs, libraries and shared resources |
| `/opt` | Optional or third-party applications |
| `/boot` | Kernel, initramfs and bootloader files |
| `/dev` | Device files |
| `/proc` | Virtual filesystem with process and kernel information |
| `/sys` | Kernel and hardware information |

Useful paths to remember:

- `/etc/passwd` — user account information.
- `/etc/group` — group information.
- `/etc/fstab` — filesystems mounted automatically at boot.
- `/etc/ssh/sshd_config` — SSH server configuration.
- `/var/log/` — location of many application and system logs.

## 1.2 Terminal, Files and Text Processing

### Navigation and file management

| Command | Purpose |
|---|---|
| `pwd` | Display current directory |
| `ls -lah` | List files, including hidden files, with readable sizes |
| `cd /etc` | Change directory |
| `cd ..` | Move to parent directory |
| `cd ~` | Go to current user's home directory |
| `find /var/log -type f -name "*.log"` | Find log files |
| `file <FILE>` | Identify file type |
| `stat <FILE>` | Display file metadata |
| `which <COMMAND>` | Show command location |
| `command -v <COMMAND>` | Locate a command or shell builtin |

File operations:

```bash
cp source.txt destination.txt
cp -a source_dir/ backup_dir/     # Preserve attributes
mv old_name new_name
mkdir -p /opt/myapp/logs
rm -i file.txt                    # Ask before deletion
tar -czf backup.tar.gz directory/ # Create compressed archive
tar -xzf backup.tar.gz            # Extract archive
```

**Warning:** `rm -rf` deletes recursively without prompting. Always verify the path before running it, especially with `sudo`.

### Reading and searching text

```bash
cat file.txt
less file.txt
head -n 20 file.txt
tail -n 50 file.txt
tail -f /var/log/syslog           # Follow a log in real time

grep -n "error" application.log
grep -Rni "failed" /var/log/
find /etc -type f -name "*.conf"
```

Useful text-processing commands:

```bash
sort names.txt
sort names.txt | uniq -c
cut -d: -f1 /etc/passwd
awk -F: '{print $1}' /etc/passwd
sed -n '1,40p' file.txt
```

### Pipes and redirection

| Syntax | Meaning |
|---|---|
| `command1 \| command2` | Send output of command 1 to command 2 |
| `>` | Write output, replacing existing file |
| `>>` | Append output to a file |
| `2>` | Redirect error output |
| `&>` | Redirect standard output and errors (Bash) |

Examples:

```bash
ip addr > network.txt
command >> output.log
command 2> errors.log
command1 | grep "error"
```

Be careful with `>`: it can immediately erase the previous contents of a file.

## 1.3 Users, Groups and Permissions

Linux permissions control who can read, modify or execute files.

### Identify the current user

```bash
whoami
id
groups
sudo -l
```

- `whoami` — current username.
- `id` — UID, primary GID and supplementary groups.
- `groups` — groups assigned to the user.
- `sudo -l` — commands the current user is allowed to run through sudo.

### User and group management

```bash
sudo adduser anna
sudo passwd anna
sudo usermod -aG sudo anna
sudo passwd -l anna
```

- `adduser` — create a user on Debian.
- `passwd` — set or change a password.
- `usermod -aG` — add a user to a supplementary group.
- `passwd -l` — lock password authentication for an account.

**Important:** Always use `-a` with `-G` when adding a supplementary group. Without `-a`, other supplementary group memberships may be replaced.

Check account and group information:

```bash
getent passwd anna
getent group sudo
```

### Read, write and execute permissions

Each file has permissions for:

- **u** — owner (user)
- **g** — group
- **o** — others

Permission values:

| Permission | Symbol | Numeric value |
|---|---|---:|
| Read | `r` | 4 |
| Write | `w` | 2 |
| Execute | `x` | 1 |
| None | `-` | 0 |

Common permission examples:

| Mode | Meaning |
|---|---|
| `755` | Owner: full; group and others: read/execute |
| `750` | Owner: full; group: read/execute; others: none |
| `700` | Only owner has access |
| `644` | Owner: read/write; others: read |
| `640` | Owner: read/write; group: read |
| `600` | Only owner can read/write |

Commands:

```bash
chmod 640 file.txt
chmod u+x script.sh
chmod -R 750 /opt/myapp
chown anna:developers file.txt
chgrp developers file.txt
```

- `chmod` — change permissions.
- `chown` — change owner and optionally group.
- `chgrp` — change group ownership.
- `-R` — apply recursively.

Avoid `chmod 777` as a workaround for permission problems. It grants everyone read, write and execute access.

### ACL and default permissions

ACLs allow permissions for additional users or groups beyond the owner/group/others model.

```bash
getfacl file.txt
setfacl -m u:anna:r file.txt
setfacl -m g:developers:rw file.txt
setfacl -x u:anna file.txt
```

Check the default permission mask:

```bash
umask
umask 027
```

`umask 027` removes write access for group and all access for others from newly created files and directories, subject to the application's requested permissions.

## 1.4 Package Management — APT and DPKG

Debian uses **APT** to manage packages and repositories. `dpkg` works at the lower package-management level.

### Update and install packages

```bash
sudo apt update
sudo apt upgrade
sudo apt install <PACKAGE>
sudo apt remove <PACKAGE>
sudo apt purge <PACKAGE>
sudo apt autoremove
```

| Command | Purpose |
|---|---|
| `apt update` | Refresh package repository indexes |
| `apt upgrade` | Upgrade installed packages |
| `apt install` | Install a package |
| `apt remove` | Remove a package, usually retaining configuration |
| `apt purge` | Remove a package and its system configuration |
| `apt autoremove` | Remove automatically installed dependencies no longer needed |

**Remember:** `apt update` does not install updates. It refreshes the available package information. Use `apt upgrade` to upgrade installed packages.

### Search and inspect packages

```bash
apt search nginx
apt show nginx
dpkg -l
dpkg -l | grep nginx
dpkg -S /usr/bin/ssh
```

- `apt search` — search available packages.
- `apt show` — display package details.
- `dpkg -l` — list installed packages and their status.
- `dpkg -S` — identify which installed package owns a file.

After important updates, check whether services need restarting and whether a reboot is required.

## 1.5 Processes and Services

A **process** is a running program. A **service** is a background component, commonly managed by `systemd`.

### Process management

```bash
ps aux
pgrep -af ssh
top
free -h
uptime
```

| Command | Purpose |
|---|---|
| `ps aux` | List running processes |
| `pgrep -af <NAME>` | Find processes by name or pattern |
| `top` | Monitor processes and CPU usage |
| `free -h` | Display RAM and swap usage |
| `uptime` | Show uptime and load average |

Stop a process:

```bash
kill -TERM <PID>
kill -KILL <PID>
```

Use `TERM` first: it requests a clean shutdown. `KILL` forces termination and should be a last resort.

### Service management with systemd

```bash
systemctl status ssh
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
sudo systemctl enable ssh
sudo systemctl enable --now ssh
sudo systemctl disable ssh
systemctl list-units --failed
```

| Command | Purpose |
|---|---|
| `status` | Show service state and recent information |
| `start` / `stop` | Start or stop a service |
| `restart` | Restart a service |
| `enable` | Configure automatic start at boot |
| `enable --now` | Enable at boot and start immediately |
| `disable` | Disable automatic start |
| `list-units --failed` | List failed systemd units |

Service names may differ between distributions and packages. For example, Debian's OpenSSH server service is normally called `ssh`.

## 1.6 Logs and Boot Diagnostics

Logs are one of the first places to investigate when a service or system behaves unexpectedly.

### journalctl

```bash
journalctl -b
journalctl -p err -b
journalctl -u ssh
journalctl -u ssh --since today
journalctl -u ssh -f
journalctl --since "1 hour ago"
```

| Command | Purpose |
|---|---|
| `journalctl -b` | Logs from the current boot |
| `journalctl -p err -b` | Error-level messages from current boot |
| `journalctl -u ssh` | Logs for the SSH service |
| `journalctl --since today` | Logs since the start of today |
| `journalctl -f` | Follow new log entries live |

### Kernel and boot information

```bash
dmesg -T
systemctl --failed
systemd-analyze blame
cat /proc/cmdline
```

- `dmesg -T` — kernel messages with readable timestamps.
- `systemctl --failed` — failed services and units.
- `systemd-analyze blame` — services' time contribution during boot.
- `/proc/cmdline` — kernel boot parameters.

### Common log locations

| Location | Content |
|---|---|
| `journalctl` | Systemd journal |
| `/var/log/syslog` | General system messages, if configured |
| `/var/log/auth.log` | Authentication and authorization events on Debian |
| `/var/log/kern.log` | Kernel messages, if configured |
| `/var/log/` | Application-specific logs |

The exact log files depend on the distribution and logging configuration. Some systems rely primarily on the systemd journal.

## 1.7 Storage, Filesystems and Mounting

### Inspect disks and filesystems

```bash
lsblk -f
blkid
sudo fdisk -l
df -h
du -sh /var/log
findmnt
```

| Command | Purpose |
|---|---|
| `lsblk -f` | List block devices, partitions and filesystems |
| `blkid` | Display filesystem identifiers and UUIDs |
| `fdisk -l` | List partition tables |
| `df -h` | Show filesystem capacity and free space |
| `du -sh <PATH>` | Show directory size |
| `findmnt` | Display mounted filesystems |

### Mounting a filesystem

```bash
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
findmnt /mnt/data
sudo umount /mnt/data
```

To configure automatic mounting at boot, edit `/etc/fstab`. Prefer a filesystem UUID rather than a device name that may change.

```bash
sudo blkid
sudo nano /etc/fstab
```

Example entry (replace the UUID and filesystem type with the actual values):

```text
UUID=<UUID>  /mnt/data  ext4  defaults  0  2
```

After editing, validate the configuration before rebooting:

```bash
sudo findmnt --verify
sudo mount -a
```

An incorrect `/etc/fstab` entry can cause boot or mounting problems.

### LVM — basic concepts

Logical Volume Manager (LVM) provides flexible storage management:

1. **PV (Physical Volume):** disk or partition prepared for LVM.
2. **VG (Volume Group):** pool of storage made from one or more PVs.
3. **LV (Logical Volume):** virtual block device allocated from a VG.

Useful inspection commands:

```bash
sudo pvs
sudo vgs
sudo lvs
```

These commands inspect LVM configuration. Creating, resizing or removing volumes requires careful planning and a verified backup.

### When a disk appears full

```bash
df -h
sudo du -xhd1 /
sudo lsof +L1
```

`df` shows filesystem usage; `du` helps locate large directories. `lsof +L1` can reveal deleted files that are still held open by running processes and therefore still consume disk space.

## 1.8 Linux Networking

### Interfaces and IP configuration

```bash
ip -br addr
ip link
ip route
ip route get 1.1.1.1
ip neigh
```

| Command | Purpose |
|---|---|
| `ip -br addr` | Brief interface and IP address overview |
| `ip link` | Interface state and link information |
| `ip route` | Display routing table |
| `ip route get <IP>` | Show the route selected for a destination |
| `ip neigh` | Display ARP/NDP neighbor cache |

### Connectivity and DNS

```bash
ping -c 4 1.1.1.1
tracepath example.com
resolvectl status
getent hosts example.com
```

- `ping` — tests IP reachability using ICMP; filtering may block replies.
- `tracepath` — helps inspect the path to a destination.
- `resolvectl status` — shows DNS configuration managed by systemd-resolved.
- `getent hosts` — resolves a hostname using the system's configured name-resolution mechanisms.

### Listening ports and network connections

```bash
ss -lntup
ss -s
curl -I https://example.com
```

- `ss -lntup` — listening TCP/UDP sockets and associated processes where permissions allow.
- `ss -s` — socket statistics.
- `curl -I` — send an HTTP HEAD request and display response headers.

### Packet capture

```bash
sudo tcpdump -ni eth0 host 10.0.0.5
```

Captures traffic on interface `eth0` involving the specified host. Replace the interface and IP with the actual values.

Packet captures can contain sensitive information. Capture only traffic you are authorized to inspect and protect the resulting files.

## 1.9 SSH and Basic Linux Hardening

SSH provides encrypted remote administration. The server configuration is normally stored in:

```text
/etc/ssh/sshd_config
```

Connect to a remote host:

```bash
ssh user@server
scp file.txt user@server:/tmp/
```

### Basic SSH security practices

- Prefer SSH keys over password authentication where operationally appropriate.
- Restrict SSH access to authorized users or management networks.
- Avoid direct root login over SSH.
- Use MFA where supported by the environment.
- Keep OpenSSH and the operating system updated.
- Monitor authentication logs.
- Do not expose SSH to the internet unnecessarily.

Before applying SSH configuration changes, validate the configuration:

```bash
sudo sshd -t
```

If validation succeeds, reload the service:

```bash
sudo systemctl reload ssh
```

**Important:** When changing SSH remotely, keep the existing session open and verify that a new session works before closing it. A bad configuration or firewall rule can lock you out.

### Firewall basics — UFW

UFW is a simplified firewall management tool commonly used on Debian-based systems.

```bash
sudo ufw status verbose
sudo ufw allow from 10.10.0.0/24 to any port 22 proto tcp
sudo ufw deny 23/tcp
```

Before enabling or changing the firewall:

1. Inspect existing rules.
2. Confirm which services must remain reachable.
3. Allow required administrative access.
4. Apply changes carefully.
5. Test access from another session or host.

Enabling a firewall remotely without allowing SSH first may disconnect you from the server.

Other Linux systems may use `nftables` directly:

```bash
sudo nft list ruleset
```

### Essential hardening checklist

- [ ] Remove or disable unnecessary accounts and services.
- [ ] Use individual user accounts; avoid shared administrator accounts.
- [ ] Apply least privilege and review `sudo` access.
- [ ] Keep packages and the kernel updated.
- [ ] Use SSH keys and restrict remote access.
- [ ] Avoid world-writable permissions such as `777`.
- [ ] Enable and review firewall rules.
- [ ] Monitor authentication and system logs.
- [ ] Keep backups and test restoration.
- [ ] Record configuration changes and maintain a rollback plan.

---

## 1.10 Quick Troubleshooting Reference

| Symptom | First checks |
|---|---|
| Server is slow | `uptime`, `top`, `free -h` |
| Disk is full | `df -h`, `du -xhd1 /`, `lsof +L1` |
| Service is unavailable | `systemctl status <SERVICE>`, `journalctl -u <SERVICE>` |
| Network interface is down | `ip -br addr`, `ip link` |
| No network connectivity | `ip route`, `ping <GATEWAY>` |
| IP works but hostname does not | `resolvectl status`, `getent hosts <HOST>` |
| Port is not reachable | `ss -lntup`, firewall rules, remote connectivity test |
| Boot or hardware issue | `journalctl -b`, `dmesg -T` |
| SSH login fails | SSH service status, `/var/log/auth.log`, firewall and SSH configuration |

### Safe administration workflow

1. **Observe:** collect the error, time, affected host and scope.
2. **Inspect:** check processes, services, logs, storage and network state.
3. **Identify:** determine the likely cause before changing configuration.
4. **Back up:** preserve relevant configuration and data.
5. **Change:** make one controlled change at a time.
6. **Verify:** test the service and check logs for new errors.
7. **Document:** record the cause, action, result and rollback procedure.
