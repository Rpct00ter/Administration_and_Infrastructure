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

# 2. Windows Administration

> **Scope:** Windows 10/11 and Windows Server. Commands are marked as **CMD** or **PowerShell** where relevant.
>
> **Important:** Run administrative commands in an elevated terminal only when required. In domain-managed environments, Group Policy (GPO) or Intune may override local settings.

## 2.1 Essential Windows Administration Tools

Many administrative tools can be opened using `Win + R`.

| Command | Tool | Purpose |
|---|---|---|
| `taskmgr` | Task Manager | Processes, performance, startup applications |
| `services.msc` | Services | Manage Windows services |
| `eventvwr.msc` | Event Viewer | Inspect system and application logs |
| `devmgmt.msc` | Device Manager | Hardware and driver management |
| `diskmgmt.msc` | Disk Management | Disks, partitions and volumes |
| `compmgmt.msc` | Computer Management | Combined administration console |
| `ncpa.cpl` | Network Connections | Network adapters and connections |
| `wf.msc` | Windows Defender Firewall | Advanced firewall rules |
| `firewall.cpl` | Windows Firewall | Basic firewall settings |
| `appwiz.cpl` | Programs and Features | Installed applications |
| `sysdm.cpl` | System Properties | Computer name, domain and system settings |
| `msinfo32` | System Information | Hardware and OS information |
| `resmon` | Resource Monitor | Detailed CPU, RAM, disk and network usage |
| `perfmon` | Performance Monitor | Performance counters |
| `taskschd.msc` | Task Scheduler | Scheduled tasks |
| `regedit` | Registry Editor | Windows registry |
| `gpedit.msc` | Local Group Policy Editor | Local policy configuration (edition-dependent) |
| `secpol.msc` | Local Security Policy | Local security settings (edition-dependent) |
| `mstsc` | Remote Desktop | Connect to another Windows computer |

Useful Windows Settings shortcut:

```text
ms-settings:windowsupdate
```

Opens Windows Update settings.

## 2.2 CMD and PowerShell Essentials

### CMD — basic system administration

| Command | Purpose |
|---|---|
| `whoami /all` | Current account, SID, groups and privileges |
| `hostname` | Computer name |
| `systeminfo` | OS, hardware and installed hotfix information |
| `ver` | Windows version |
| `tasklist` | List running processes |
| `tasklist /svc` | Processes and associated services |
| `driverquery /v` | Detailed driver information |
| `net user` | List local users |
| `net localgroup` | List local groups |
| `net localgroup Administrators` | Show local Administrators group members |

Examples:

```cmd
hostname
systeminfo
whoami /all
tasklist /svc
net user
net localgroup Administrators
```

Terminate a process by PID:

```cmd
taskkill /PID 1234 /T
```

- `/PID` — process ID.
- `/T` — terminate the process and its child processes.
- Add `/F` to force termination when necessary.

Always verify the PID before terminating a process.

### PowerShell — administration and automation

PowerShell uses objects and pipelines, making it especially useful for filtering and managing system information.

| Command | Purpose |
|---|---|
| `Get-Command` | Find available commands |
| `Get-Help` | Display command help |
| `Get-ComputerInfo` | System information |
| `Get-HotFix` | Installed Windows updates/hotfixes |
| `Get-Service` | List services |
| `Get-Process` | List processes |
| `Get-LocalUser` | List local accounts |
| `Get-LocalGroup` | List local groups |
| `Get-Volume` | List volumes |
| `Get-Disk` | List disks |
| `Get-Partition` | List partitions |

Examples:

```powershell
Get-Command *net*
Get-Help Get-Service -Examples

Get-ComputerInfo
Get-HotFix

Get-Process | Sort-Object CPU -Descending |
    Select-Object -First 10

Get-Service | Where-Object Status -ne 'Running'
```

PowerShell pipeline operators:

| Operator | Purpose |
|---|---|
| `\|` | Pass objects to another command |
| `Where-Object` | Filter objects |
| `Select-Object` | Select properties or limit results |
| `Sort-Object` | Sort results |
| `Format-Table` | Display results as a table |

Example:

```powershell
Get-Service |
    Where-Object Status -eq 'Running' |
    Sort-Object Name |
    Format-Table Name, Status
```

## 2.3 Users, Groups and Permissions

### Local account management

```powershell
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember -Group 'Administrators'
```

Create a local user:

```powershell
$password = Read-Host "Enter password" -AsSecureString

New-LocalUser -Name "anna" `
    -Password $password `
    -FullName "Anna User"
```

Add the user to the local Administrators group:

```powershell
Add-LocalGroupMember -Group 'Administrators' -Member 'anna'
```

Remove a user from a group:

```powershell
Remove-LocalGroupMember -Group 'Administrators' -Member 'anna'
```

Disable an account:

```powershell
Disable-LocalUser -Name 'anna'
```

Enable it again:

```powershell
Enable-LocalUser -Name 'anna'
```

**Security principles:**

- Use individual accounts rather than shared administrator accounts.
- Give users only the permissions they need (least privilege).
- Limit membership of the local Administrators group.
- Disable accounts that are no longer required.
- Use MFA where available.
- Review privileged group membership regularly.

### NTFS permissions

Windows uses NTFS Access Control Lists (ACLs) to control access to files and directories.

Common permissions:

| Permission | Meaning |
|---|---|
| Full Control | Read, write, modify permissions and take ownership |
| Modify | Read, write, execute and delete |
| Read & Execute | Read files and execute programs |
| Read | View file contents |
| Write | Create or modify files |

Inspect permissions:

```cmd
icacls C:\Data
```

Grant read access:

```cmd
icacls C:\Data /grant anna:(R)
```

Grant modify access:

```cmd
icacls C:\Data /grant anna:(M)
```

Apply permissions recursively:

```cmd
icacls C:\Data /grant anna:(OI)(CI)M /T
```

- `(OI)` — permission inheritance by files.
- `(CI)` — permission inheritance by directories.
- `/T` — process files and subdirectories recursively.

Remove an explicit permission entry:

```cmd
icacls C:\Data /remove anna
```

**Important:** NTFS permissions can be combined with share permissions when accessing a network share. Effective access is determined by both. Avoid broad permissions such as `Everyone: Full Control`.

## 2.4 Processes, Services and Scheduled Tasks

### Process management

PowerShell:

```powershell
Get-Process

Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 10

Get-Process -Name notepad
Stop-Process -Name notepad
```

CMD:

```cmd
tasklist
tasklist /FI "IMAGENAME eq notepad.exe"
taskkill /IM notepad.exe
```

Use `Stop-Process` or `taskkill` carefully. Terminating a process may cause unsaved data loss or interrupt a service.

### Windows services

PowerShell:

```powershell
Get-Service

Get-Service -Name Spooler
Start-Service -Name Spooler
Stop-Service -Name Spooler
Restart-Service -Name Spooler
```

Inspect service configuration:

```powershell
Get-CimInstance Win32_Service |
    Select-Object Name, State, StartMode, StartName, PathName
```

CMD:

```cmd
sc query
sc query Spooler
sc qc Spooler
```

Services console:

```text
services.msc
```

Common service properties:

- **Name:** internal service identifier.
- **Display name:** human-readable service name.
- **Status:** running, stopped or another state.
- **Startup type:** automatic, manual or disabled.
- **Log On As:** account under which the service runs.

Do not disable services simply because their purpose is unclear. First identify dependencies and business impact.

### Scheduled tasks

Open Task Scheduler:

```text
taskschd.msc
```

PowerShell:

```powershell
Get-ScheduledTask
Get-ScheduledTask -TaskPath '\Microsoft\Windows\'
```

CMD:

```cmd
schtasks /query /fo LIST /v
```

Scheduled tasks are commonly used for maintenance, backups, scripts and application jobs. When troubleshooting, inspect the task's trigger, action, execution account and last result.

## 2.5 Event Viewer and System Logs

Event Viewer is one of the most important Windows troubleshooting tools.

Open it:

```text
eventvwr.msc
```

Main log categories:

| Log | What to investigate |
|---|---|
| System | Drivers, services, boot and hardware errors |
| Application | Application crashes and errors |
| Security | Logons, account activity and security auditing |
| Setup | Installation and update events |

### PowerShell event inspection

Read recent System events:

```powershell
Get-WinEvent -LogName System -MaxEvents 50
```

Read recent Application events:

```powershell
Get-WinEvent -LogName Application -MaxEvents 50
```

Filter Security events:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624,4625
} -MaxEvents 30
```

Important Security Event IDs:

| Event ID | Meaning |
|---:|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4648 | Logon using explicitly supplied credentials |
| 4672 | Special privileges assigned to a new logon |
| 4688 | New process created (if auditing is configured) |
| 4697 | Service installed |
| 4698 | Scheduled task created |
| 4720 | User account created |
| 4728 | Member added to a global security group |
| 4732 | Member added to a local security group |

**Note:** Events appear only when the relevant auditing is enabled and the log has retained the data. An absent event does not prove that an action did not happen.

### Additional diagnostic tools

| Tool | Command | Purpose |
|---|---|---|
| Reliability Monitor | `perfmon /rel` | History of crashes and system stability |
| Resource Monitor | `resmon` | Detailed resource usage |
| Performance Monitor | `perfmon` | Performance counters and monitoring |
| Device Manager | `devmgmt.msc` | Driver and hardware problems |

## 2.6 Disk and Storage Management

### Graphical tools

```text
diskmgmt.msc
```

Disk Management allows administrators to:

- Inspect physical disks and partitions.
- Create and format volumes.
- Assign drive letters.
- Extend or shrink supported volumes.
- Inspect disk status.

### PowerShell storage commands

```powershell
Get-Disk
Get-Partition
Get-Volume
```

Inspect a particular disk:

```powershell
Get-Disk -Number 0 | Format-List *
```

Inspect free space:

```powershell
Get-Volume |
    Select-Object DriveLetter, FileSystem,
        Size, SizeRemaining
```

### Disk-related troubleshooting

Check:

1. Whether the disk is detected.
2. Whether it is online.
3. Whether the partition exists.
4. Whether the filesystem is healthy.
5. Whether a drive letter or mount point is assigned.
6. Whether the volume has free space.

**Warning:** Initializing, formatting, repartitioning or changing disk layouts can destroy data. Verify the selected disk and ensure a backup exists before making changes.

## 2.7 Windows Networking

### CMD — network configuration and diagnostics

```cmd
ipconfig
ipconfig /all
ipconfig /flushdns
ipconfig /release
ipconfig /renew

ping -n 4 1.1.1.1
tracert 1.1.1.1
pathping 1.1.1.1
nslookup example.com

netstat -ano
arp -a
```

| Command | Purpose |
|---|---|
| `ipconfig /all` | Full IP configuration |
| `ipconfig /flushdns` | Clear local DNS resolver cache |
| `ipconfig /release` | Release DHCP-assigned IPv4 address |
| `ipconfig /renew` | Request a new DHCP IPv4 address |
| `ping` | Test IP reachability using ICMP |
| `tracert` | Show the route toward a destination |
| `pathping` | Route and packet-loss diagnostics |
| `nslookup` | Query DNS |
| `netstat -ano` | Connections/listening ports and associated PIDs |
| `arp -a` | Display IPv4 ARP cache |

**Caution:** `ipconfig /release` temporarily removes the DHCP-assigned IPv4 configuration. Do not use it over a remote connection unless you have an alternative way to access the machine.

### PowerShell — network diagnostics

```powershell
Get-NetIPConfiguration
Get-NetAdapter
Get-NetRoute
Get-NetNeighbor
```

Test DNS resolution:

```powershell
Resolve-DnsName example.com
```

Test whether a TCP port is reachable:

```powershell
Test-NetConnection server.example.com -Port 443
```

Inspect listening TCP ports:

```powershell
Get-NetTCPConnection -State Listen
```

Display process IDs owning TCP connections:

```powershell
Get-NetTCPConnection |
    Select-Object State, LocalAddress, LocalPort,
        RemoteAddress, RemotePort, OwningProcess
```

### Basic network troubleshooting sequence

1. Check the network adapter:

   ```powershell
   Get-NetAdapter
   ```

2. Check IP address, gateway and DNS:

   ```powershell
   Get-NetIPConfiguration
   ```

3. Test the local TCP/IP stack and network reachability:

   ```cmd
   ping 127.0.0.1
   ping <DEFAULT_GATEWAY>
   ```

4. Test external IP connectivity:

   ```cmd
   ping 1.1.1.1
   ```

5. Test DNS resolution:

   ```powershell
   Resolve-DnsName example.com
   ```

6. Test the required service port:

   ```powershell
   Test-NetConnection <HOST> -Port <PORT>
   ```

A failed ping alone does not prove a host is offline: ICMP may be blocked by a firewall.

## 2.8 Windows System Repair and Maintenance

### System File Checker (SFC)

Checks protected Windows system files and attempts to repair corrupted files.

Run in an elevated CMD or PowerShell:

```cmd
sfc /scannow
```

### DISM

Repairs the Windows component store, which can be used as a source for system file repairs.

```cmd
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
DISM /Online /Cleanup-Image /RestoreHealth
```

Typical repair sequence:

```cmd
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow
```

Restart if required and verify whether the original problem is resolved.

### Updates and installed patches

PowerShell:

```powershell
Get-HotFix
```

Open Windows Update:

```text
ms-settings:windowsupdate
```

Good patch-management practice:

- Install security updates regularly.
- Follow organizational maintenance windows.
- Check compatibility and known issues for critical systems.
- Plan required restarts.
- Verify services and applications after updates.
- Keep a rollback or recovery plan for important systems.

## 2.9 Windows Firewall and Basic Hardening

Open advanced firewall management:

```text
wf.msc
```

Inspect firewall profiles:

```powershell
Get-NetFirewallProfile |
    Format-Table Name, Enabled, DefaultInboundAction
```

List enabled rules:

```powershell
Get-NetFirewallRule -Enabled True |
    Select-Object DisplayName, Direction, Action
```

### Basic firewall principles

- Keep the firewall enabled on all applicable profiles.
- Block unsolicited inbound traffic by default where appropriate.
- Allow only required services and ports.
- Restrict administrative access to trusted networks.
- Avoid broad rules that expose services unnecessarily.
- Review and document firewall exceptions.

### Essential Windows security checklist

- [ ] Keep Windows and applications patched.
- [ ] Use standard accounts for everyday work.
- [ ] Restrict local Administrators membership.
- [ ] Enable MFA where available.
- [ ] Keep Microsoft Defender or the approved endpoint protection active.
- [ ] Enable Windows Firewall.
- [ ] Disable unnecessary services and remote access.
- [ ] Secure RDP and restrict access to trusted networks.
- [ ] Review local users and privileged groups.
- [ ] Monitor Security and System event logs.
- [ ] Maintain tested backups.
- [ ] Follow organizational GPO/Intune policies.

## 2.10 Quick Troubleshooting Reference

| Symptom | First checks |
|---|---|
| Computer is slow | Task Manager, `resmon`, `Get-Process` |
| High CPU usage | Sort processes by CPU usage |
| Service unavailable | `Get-Service`, `services.msc`, Event Viewer |
| Application crashes | Application log, Reliability Monitor |
| No network connection | `ipconfig /all`, `Get-NetAdapter`, gateway test |
| IP works but hostname fails | `Resolve-DnsName`, `ipconfig /flushdns` |
| Remote port unavailable | `Test-NetConnection <HOST> -Port <PORT>` |
| Disk is full | `Get-Volume`, Disk Management |
| Windows system files corrupted | DISM, then `sfc /scannow` |
| Suspicious login activity | Security log, Event IDs 4624 and 4625 |
| Unexpected process | Task Manager, `Get-Process`, process path and signature |

### Safe administration workflow

1. **Identify:** record the symptom, affected user/system and time.
2. **Inspect:** check processes, services, logs, storage and network state.
3. **Check permissions:** determine whether the task requires elevation.
4. **Back up:** preserve relevant configuration before making changes.
5. **Change:** make one controlled change at a time.
6. **Verify:** test the service or function and inspect relevant logs.
7. **Document:** record the action, result and rollback procedure.

# 3. Windows Server and Active Directory

> **Scope:** Windows Server administration with a focus on Active Directory Domain Services (AD DS), domain accounts, Group Policy, DNS and DHCP.
>
> **Important:** Active Directory changes can affect authentication and access across an entire organization. Perform administrative changes with appropriate permissions, backups and a rollback plan.

## 3.1 Windows Server Roles and Features

Windows Server provides roles and features that deliver network services.

A **role** provides a major server function. A **feature** adds supporting functionality.

Common roles:

| Role | Purpose |
|---|---|
| AD DS | Centralized identity and directory management |
| DNS Server | Name resolution for hosts and services |
| DHCP Server | Automatic IP address configuration |
| File and Storage Services | File shares and storage management |
| Hyper-V | Virtual machine hosting |
| Web Server (IIS) | Hosting websites and web applications |
| Remote Desktop Services | Remote desktop and application delivery |

Inspect installed roles and features (PowerShell, elevated):

```powershell
Get-WindowsFeature
```

Install a role:

```powershell
Install-WindowsFeature -Name DNS -IncludeManagementTools
```

For AD DS, installing the role is only the first step. The server must also be configured as a domain controller or added to an existing domain.

## 3.2 Active Directory — Core Concepts

**Active Directory Domain Services (AD DS)** is Microsoft's directory service for centrally managing identities, computers and resources in a Windows domain.

It stores objects such as users, groups, computers and shared resources. It supports authentication, access control and policy-based administration. [Microsoft Learn — AD DS overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-domain-services)

### Main components

| Component | Meaning |
|---|---|
| Forest | Top-level AD structure; contains one or more domains |
| Domain | Administrative and replication unit containing directory objects |
| Organizational Unit (OU) | Container used to organize objects and apply/delegate administration |
| Domain Controller (DC) | Server running AD DS that authenticates users and stores directory data |
| Global Catalog (GC) | Helps locate objects across the forest |
| Site | Represents network topology and helps manage replication between locations |
| Trust | Relationship that allows authentication across domains or forests |

Typical hierarchy:

```text
Forest: company.local
│
├── Domain: company.local
│   ├── OU: Users
│   │   ├── User: john
│   │   └── User: anna
│   │
│   ├── OU: Computers
│   │   ├── PC-001
│   │   └── PC-002
│   │
│   ├── OU: Servers
│   │   └── SRV-01
│   │
│   └── OU: Groups
│       ├── IT-Admins
│       └── Finance
```

**Remember:**
- A forest can contain multiple domains.
- A domain contains users, computers, groups and other objects.
- OUs organize objects and provide a scope for Group Policy and delegated administration.
- A domain controller provides directory and authentication services.
- Most organizations can operate with a single forest and domain; multiple domains should have a clear administrative or technical reason.

[Microsoft Learn — AD DS logical structure](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/designing-the-logical-structure)

## 3.3 Domain Controllers and Domain Setup

A **Domain Controller (DC)** hosts AD DS and provides services such as:
- User and computer authentication.
- Directory lookups.
- Group Policy processing support.
- Replication with other domain controllers.

A production domain should normally have more than one DC to improve availability.

### Basic deployment sequence

1. Install Windows Server.
2. Configure a static IP address.
3. Configure the correct DNS settings.
4. Set a suitable computer name.
5. Install the AD DS server role.
6. Promote the server to a domain controller.
7. Create a new forest/domain or add the server to an existing domain.
8. Verify DNS, authentication and replication.

Install AD DS role:

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

The server can then be promoted using Server Manager or the AD DS PowerShell deployment cmdlets.

Example: create a new forest (lab/example environment):

```powershell
Install-ADDSForest -DomainName "company.local"
```

This operation requires planning, appropriate credentials and a restart. Do not run it on an existing production server without confirming the intended domain design.

### Check domain controller health

```cmd
dcdiag
dcdiag /v
```

`dcdiag` checks important domain controller functions and reports potential problems.

Check which DC authenticated the current session:

```cmd
echo %LOGONSERVER%
```

## 3.4 Users, Groups and Computers

Active Directory objects are commonly managed through:

- **Active Directory Users and Computers:** `dsa.msc`
- **Active Directory Administrative Center:** `dsac.exe`
- **PowerShell ActiveDirectory module**

### Common user administration tasks

PowerShell examples (requires the ActiveDirectory module and suitable permissions):

```powershell
Import-Module ActiveDirectory

Get-ADUser -Filter *
Get-ADUser john -Properties *
```

Create a user:

```powershell
New-ADUser `
    -Name "John Smith" `
    -SamAccountName "jsmith" `
    -UserPrincipalName "jsmith@company.local" `
    -Path "OU=Users,DC=company,DC=local" `
    -Enabled $true
```

Set a password:

```powershell
Set-ADAccountPassword jsmith -Reset
```

Enable or disable an account:

```powershell
Enable-ADAccount jsmith
Disable-ADAccount jsmith
```

Unlock an account:

```powershell
Unlock-ADAccount jsmith
```

Inspect account properties:

```powershell
Get-ADUser jsmith -Properties Enabled,LockedOut,LastLogonDate
```

### Group management

Groups simplify permission management. Assign permissions to groups rather than individually to many users.

Common group scopes:

| Scope | Main use |
|---|---|
| Global | Organize users from the same domain |
| Domain Local | Assign permissions to resources in a domain |
| Universal | Group members from multiple domains in a forest |

Create a security group:

```powershell
New-ADGroup `
    -Name "IT-Support" `
    -GroupScope Global `
    -GroupCategory Security `
    -Path "OU=Groups,DC=company,DC=local"
```

Add a user:

```powershell
Add-ADGroupMember -Identity "IT-Support" -Members "jsmith"
```

Check group membership:

```powershell
Get-ADGroupMember "IT-Support"
```

**Practical rule:** Use groups to assign access to file shares, applications and other resources. Avoid assigning permissions individually to each user.

## 3.5 Group Policy (GPO)

**Group Policy** centrally manages Windows configuration for users and computers in an Active Directory environment.

Examples of settings controlled by GPO:

- Password and account policies.
- Windows Defender Firewall.
- Windows Update configuration.
- Desktop and control panel restrictions.
- Security settings.
- Scripts and mapped drives.
- Software deployment and configuration.

### Group Policy basics

Open Group Policy Management:

```text
gpmc.msc
```

A GPO can be linked to:
- A site.
- A domain.
- An OU.

Typical processing order:

1. Local policy.
2. Site policy.
3. Domain policy.
4. Parent OU policy.
5. Child OU policy.

This is commonly called **LSDOU**. Inheritance, link order, enforcement and security filtering can affect the final result.

### Update and inspect policy

Force a policy refresh:

```cmd
gpupdate /force
```

Display applied policies:

```cmd
gpresult /r
```

Generate a detailed HTML report:

```cmd
gpresult /h C:\Temp\gpo-report.html
```

**Troubleshooting tip:** If a GPO does not apply, check the computer/user's OU, security filtering, inheritance, replication and whether the setting is configured under Computer Configuration or User Configuration.

## 3.6 DNS and DHCP in a Domain

DNS and DHCP are essential network services in many Active Directory environments.

### DNS — Domain Name System

DNS resolves names to IP addresses. Active Directory also relies on DNS records to locate domain controllers and services.

Important records:

| Record | Purpose |
|---|---|
| A | Hostname to IPv4 address |
| AAAA | Hostname to IPv6 address |
| CNAME | Alias for another hostname |
| MX | Mail server |
| PTR | Reverse lookup: IP to hostname |
| SRV | Locates services such as domain controllers |

AD domain clients use DNS SRV records to locate domain services. Incorrect DNS configuration is a common cause of domain join and authentication problems.

Useful commands:

```powershell
Resolve-DnsName company.local
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.company.local
```

```cmd
nslookup company.local
ipconfig /all
ipconfig /flushdns
```

### DHCP — Dynamic Host Configuration Protocol

DHCP automatically provides network configuration to clients:

- IP address.
- Subnet mask.
- Default gateway.
- DNS server addresses.
- Lease duration and other options.

The DHCPv4 process is known as **DORA**:

1. Discover — client searches for a DHCP server.
2. Offer — server offers an IP address.
3. Request — client requests the offered address.
4. Acknowledge — server confirms the lease.

Useful DHCP checks:

- Is the DHCP server running and authorized where required?
- Is the scope active?
- Are addresses available?
- Are gateway and DNS options correct?
- Is DHCP relay/IP helper configured between VLANs?
- Is the client connected to the correct network?

DHCP does not have to run on a domain controller; it can be hosted on a separate server or network device.

## 3.7 Kerberos, LDAP and Authentication

### Kerberos

Kerberos is a major authentication protocol used in Active Directory domains.

Key points:
- Uses tickets rather than repeatedly sending passwords to services.
- Relies on the Key Distribution Center (KDC), normally hosted by domain controllers.
- Uses time-sensitive authentication.
- DNS and accurate time synchronization are important for reliable operation.

A significant time difference between a client and domain controller can cause Kerberos authentication failures.

### LDAP

LDAP (Lightweight Directory Access Protocol) is used to query and manage directory information.

| Port | Service |
|---:|---|
| TCP/UDP 389 | LDAP |
| TCP 636 | LDAPS (LDAP over TLS) |
| TCP/UDP 88 | Kerberos |
| TCP/UDP 53 | DNS |
| UDP 123 | NTP/time synchronization |
| TCP 445 | SMB |

Use secure LDAP transport where supported and required by organizational policy. Do not expose directory services unnecessarily to untrusted networks.

## 3.8 Domain Join and Replication

### Join a Windows computer to a domain

Before joining:
- Ensure the computer can reach a domain controller.
- Configure DNS to use the organization's AD DNS servers.
- Verify network connectivity and time synchronization.
- Have authorized domain credentials.

PowerShell (run with appropriate privileges):

```powershell
Add-Computer -DomainName "company.local" -Credential "COMPANY\Administrator" -Restart
```

The computer restarts after joining.

Check domain membership:

```powershell
Get-CimInstance Win32_ComputerSystem |
    Select-Object Name, Domain, PartOfDomain
```

### Active Directory replication

When changes are made on one domain controller, AD replication distributes directory changes to other domain controllers.

Check replication summary:

```cmd
repadmin /replsummary
```

Show replication partners and status:

```cmd
repadmin /showrepl
```

Trigger replication (use only when appropriate):

```cmd
repadmin /syncall /AdeP
```

Replication problems can cause:
- Users or groups appearing differently on different DCs.
- Delayed account or password changes.
- Authentication problems.
- Inconsistent Group Policy or directory information.

Do not treat forced replication as a fix for an underlying DNS, connectivity, time or DC health problem.

## 3.9 Essential Active Directory Troubleshooting

| Symptom | First checks |
|---|---|
| Cannot join domain | DNS settings, DC reachability, time, credentials |
| User cannot log in | Account enabled/locked, password, DC connectivity, time |
| Account repeatedly locks | Security logs, scheduled tasks, saved credentials, services |
| GPO not applying | `gpresult`, OU location, filtering, replication |
| Cannot find domain controller | DNS SRV records, DNS server, network connectivity |
| Changes missing on another DC | `repadmin /replsummary`, `repadmin /showrepl` |
| Kerberos authentication fails | Time synchronization, DNS, account/SPN configuration |
| Slow domain logon | DNS, DC availability, GPO processing, network path |

### Essential diagnostic commands

```cmd
ipconfig /all
nslookup company.local
gpupdate /force
gpresult /r
dcdiag
repadmin /replsummary
repadmin /showrepl
```

PowerShell:

```powershell
Get-ADDomain
Get-ADForest
Get-ADDomainController -Filter *
Get-ADUser -Filter * -SearchBase "OU=Users,DC=company,DC=local"
Get-ADGroupMember "Domain Admins"
```

## 3.10 Active Directory Security Checklist

- [ ] Use separate privileged accounts for administration.
- [ ] Minimize membership of Domain Admins and other privileged groups.
- [ ] Use MFA for administrative access where supported.
- [ ] Keep domain controllers patched and securely configured.
- [ ] Restrict network access to domain controllers.
- [ ] Use secure LDAP configurations where required.
- [ ] Monitor successful and failed logons.
- [ ] Review inactive accounts and stale computer objects.
- [ ] Apply security settings through controlled GPOs.
- [ ] Maintain and test AD-aware backups and recovery procedures.
- [ ] Monitor DNS, time synchronization and replication health.
- [ ] Protect domain controller access and administrative credentials.

**Remember:** Domain controllers are critical infrastructure. Compromise of a domain controller can give an attacker control over domain authentication and authorization.

---

## Quick Reference

| Task | Tool / Command |
|---|---|
| AD Users and Computers | `dsa.msc` |
| Group Policy Management | `gpmc.msc` |
| DNS Manager | `dnsmgmt.msc` |
| DHCP Manager | `dhcpmgmt.msc` |
| Check domain controller health | `dcdiag` |
| Check replication | `repadmin /replsummary` |
| Force policy update | `gpupdate /force` |
| Show applied policies | `gpresult /r` |
| List domain controllers | `Get-ADDomainController -Filter *` |
| List domain users | `Get-ADUser -Filter *` |
| Check current logon server | `echo %LOGONSERVER%` |

