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

# 4. Networking Fundamentals and Protocols

## 4.1 OSI and TCP/IP Models

The **OSI model** describes network communication in seven layers. It is useful for understanding protocols and troubleshooting.

| Layer | Name | Main function | Examples |
|---|---|---|---|
| 7 | Application | Network services used by applications | HTTP, DNS, SMTP, SSH |
| 6 | Presentation | Data format, encryption, compression | TLS, encoding |
| 5 | Session | Establishes and manages sessions | Session control |
| 4 | Transport | End-to-end communication, ports | TCP, UDP |
| 3 | Network | IP addressing and routing | IPv4, IPv6, ICMP |
| 2 | Data Link | Local delivery using MAC addresses | Ethernet, Wi-Fi, VLAN |
| 1 | Physical | Transmits raw bits | Copper, fiber, radio |

### TCP/IP model

| TCP/IP layer | OSI equivalent | Examples |
|---|---|---|
| Application | 5–7 | HTTP, DNS, SSH, DHCP, TLS |
| Transport | 4 | TCP, UDP |
| Internet | 3 | IPv4, IPv6, ICMP |
| Network Access | 1–2 | Ethernet, Wi-Fi, ARP |

### Encapsulation

When data is sent, each layer adds information required for delivery:

`Application data → TCP segment / UDP datagram → IP packet → Ethernet frame → bits`

At the destination, the process is reversed (**decapsulation**).

Important:
- IP addresses identify source and destination across networks.
- MAC addresses identify devices on the local network segment.
- MAC addresses normally change at every router hop.
- IP addresses usually remain the same end-to-end, unless NAT modifies them.

## 4.2 TCP and UDP

| TCP | UDP |
|---|---|
| Connection-oriented | Connectionless |
| Reliable delivery | No delivery guarantee |
| Uses acknowledgements and retransmissions | No built-in retransmission |
| Maintains connection state | Lower overhead |
| Used where reliable delivery matters | Used where low latency or simple transport matters |

### TCP three-way handshake

1. **SYN** — client requests a connection.
2. **SYN-ACK** — server acknowledges and responds.
3. **ACK** — client confirms; connection is established.

Other important TCP flags:
- `FIN` — initiates normal connection closure.
- `RST` — resets a connection, often because of an error or rejection.
- `ACK` — acknowledges received data.

Use TCP when reliable, ordered delivery is required. UDP is common for DNS queries, voice/video traffic and other latency-sensitive services.

## 4.3 Ethernet and MAC Addresses

Ethernet operates mainly at OSI Layer 2.

A **MAC address** is normally a 48-bit hardware address represented in hexadecimal, for example:

`00:1A:2B:3C:4D:5E`

The first 24 bits normally identify the manufacturer (OUI); the remaining bits identify the interface.

### Types of Ethernet communication

| Type | Meaning |
|---|---|
| Unicast | One sender to one destination |
| Multicast | One sender to a selected group |
| Broadcast | One sender to all devices in the local broadcast domain |

Ethernet broadcast MAC address:

`FF:FF:FF:FF:FF:FF`

### ARP — Address Resolution Protocol

ARP maps an IPv4 address to a MAC address on the local network.

Example:
1. Host needs to send data to `192.168.1.1`.
2. It checks its ARP cache.
3. If no entry exists, it broadcasts an ARP Request.
4. The device owning `192.168.1.1` replies with its MAC address.
5. The sender stores the result in its ARP cache.

ARP does not cross routers.

**Gratuitous ARP**: a host announces or checks its own IP address. It can help detect duplicate IP addresses and update neighbouring ARP caches.

Useful commands:

```bash
ip neigh
arp -a
```

```powershell
arp -a
Get-NetNeighbor
```

## 4.4 IPv4 Addressing and Subnetting

IPv4 uses **32-bit addresses**, written as four decimal octets.

Example:

`192.168.1.10`

An IP address consists of:
- Network portion — identifies the network.
- Host portion — identifies a device within that network.

The subnet mask or CIDR prefix determines how many bits belong to the network.

### Common subnet masks

| CIDR | Subnet mask | Total addresses | Usable hosts* |
|---|---|---:|---:|
| /24 | 255.255.255.0 | 256 | 254 |
| /25 | 255.255.255.128 | 128 | 126 |
| /26 | 255.255.255.192 | 64 | 62 |
| /27 | 255.255.255.224 | 32 | 30 |
| /28 | 255.255.255.240 | 16 | 14 |
| /29 | 255.255.255.248 | 8 | 6 |
| /30 | 255.255.255.252 | 4 | 2 |
| /31 | 255.255.255.254 | 2 | 2 (point-to-point) |
| /32 | 255.255.255.255 | 1 | 1 address |

*For ordinary IPv4 subnets, usable hosts are generally total addresses minus network and broadcast addresses. `/31` and `/32` are exceptions.*

Formula:

- Number of addresses: `2^(32 - prefix)`
- Typical usable hosts: `2^(32 - prefix) - 2`

### Example: 192.168.10.77/26

A `/26` subnet has 64 addresses per block.

| Parameter | Result |
|---|---|
| Network address | 192.168.10.64 |
| First usable host | 192.168.10.65 |
| Last usable host | 192.168.10.126 |
| Broadcast address | 192.168.10.127 |
| Next subnet | 192.168.10.128/26 |

**Subnetting procedure:**
1. Identify the CIDR prefix.
2. Calculate the number of addresses.
3. Determine the subnet block size.
4. Find the network and broadcast addresses.
5. Identify the usable host range.
6. Check that the host and default gateway belong to the same subnet.

### Important IPv4 ranges

| Range | Purpose |
|---|---|
| `10.0.0.0/8` | Private network |
| `172.16.0.0/12` | Private network |
| `192.168.0.0/16` | Private network |
| `127.0.0.0/8` | Loopback |
| `169.254.0.0/16` | Link-local / APIPA |
| `0.0.0.0/0` | Default route |
| `224.0.0.0/4` | Multicast |

A `169.254.x.x` address on a typical client often indicates that it did not obtain an IPv4 address from DHCP.

## 4.5 IPv6 Basics

IPv6 uses **128-bit addresses**, written in hexadecimal.

Example:

`2001:db8:1::10`

Common address types:

| Type | Range / example | Purpose |
|---|---|---|
| Global Unicast | `2000::/3` | Globally routable |
| Link-local | `FE80::/10` | Communication on the local link |
| Unique Local Address | `FC00::/7` | Private/internal addressing |
| Loopback | `::1` | Local host |
| Multicast | `FF00::/8` | Communication with a group |

Important differences:
- IPv6 has no broadcast; it uses multicast.
- `/64` is common for IPv6 LAN subnets.
- SLAAC allows devices to configure addresses using Router Advertisements.
- DHCPv6 can also provide address or configuration information.
- NDP replaces several IPv4 neighbour-discovery functions, including ARP.

Useful commands:

```bash
ip -6 addr
ip -6 route
ping -6 2001:db8::1
traceroute -6 example.com
```

Do not block ICMPv6 indiscriminately. It is important for IPv6 neighbour discovery and Path MTU Discovery.

## 4.6 DNS, DHCP and NAT

### DNS — Domain Name System

DNS translates domain names into IP addresses.

Common record types:

| Record | Purpose |
|---|---|
| A | Name to IPv4 address |
| AAAA | Name to IPv6 address |
| CNAME | Alias to another name |
| MX | Mail server |
| NS | Authoritative name server |
| PTR | Reverse lookup: IP to name |
| TXT | Text data, often used for verification and email security |

Useful commands:

```bash
dig example.com
dig example.com A
dig -x 192.168.1.10
nslookup example.com
```

```powershell
Resolve-DnsName example.com
Clear-DnsClientCache
```

If DNS fails, check:
- Whether the client has the correct DNS server configured.
- Whether the DNS server is reachable.
- Whether the record exists and is correct.
- Whether the problem affects one name or all names.

### DHCP — Dynamic Host Configuration Protocol

DHCP automatically provides clients with network configuration, usually:
- IP address
- Subnet mask
- Default gateway
- DNS server
- Lease duration

DHCPv4 uses the **DORA** process:

1. **Discover** — client searches for a DHCP server.
2. **Offer** — server offers an address.
3. **Request** — client requests the offered address.
4. **Acknowledgement** — server confirms the lease.

Common ports:
- DHCP server: UDP 67
- DHCP client: UDP 68

If clients are in a different VLAN from the DHCP server, a DHCP relay (often called an IP helper) may be required.

### NAT — Network Address Translation

NAT translates IP addresses, commonly between private internal addresses and public addresses.

- **Static NAT** — fixed one-to-one address mapping.
- **Dynamic NAT** — maps private addresses to addresses from a public pool.
- **PAT / NAT overload** — allows multiple internal devices to share one public IP by using different port numbers.

NAT is not a replacement for a firewall. It does not automatically provide complete network security.

## 4.7 Default Gateway and Routing

A **default gateway** is the router or Layer 3 interface a host uses to reach destinations outside its local subnet.

Basic decision:
- Destination in the local subnet → send directly to the destination using its MAC address.
- Destination outside the local subnet → send the frame to the default gateway's MAC address.

Routers forward packets using their routing tables. They select a route based on the destination IP and the most specific matching prefix.

A default route is commonly written as:

`0.0.0.0/0` (IPv4)  
`::/0` (IPv6)

Useful commands:

```bash
ip route
ip route get 8.8.8.8
```

```powershell
route print
Get-NetRoute
```

## 4.8 Important Network Ports and Protocols

| Service / protocol | Port | Transport |
|---|---:|---|
| FTP control | 21 | TCP |
| SSH / SFTP | 22 | TCP |
| Telnet | 23 | TCP |
| SMTP | 25 | TCP |
| DNS | 53 | UDP/TCP |
| DHCP server | 67 | UDP |
| DHCP client | 68 | UDP |
| HTTP | 80 | TCP |
| Kerberos | 88 | TCP/UDP |
| POP3 | 110 | TCP |
| NTP | 123 | UDP |
| IMAP | 143 | TCP |
| SNMP | 161/162 | UDP |
| LDAP | 389 | TCP/UDP |
| HTTPS | 443 | TCP |
| SMB | 445 | TCP |
| SMTPS / submission | 465 / 587 | TCP |
| LDAPS | 636 | TCP |
| IMAPS | 993 | TCP |
| POP3S | 995 | TCP |
| RDP | 3389 | TCP/UDP |

Practical security notes:
- Prefer SSH over Telnet.
- Prefer HTTPS over HTTP.
- Use SNMPv3 instead of older SNMP versions where possible.
- Restrict SMB and RDP to trusted networks and authorized users.
- Do not expose RDP directly to the Internet.
- Use encrypted LDAP (LDAPS or properly configured LDAP with TLS).

## 4.9 Quick Reference

| Concept | Remember |
|---|---|
| MAC address | Local Layer 2 hardware address |
| IP address | Logical Layer 3 address |
| TCP | Reliable, connection-oriented transport |
| UDP | Connectionless transport with lower overhead |
| ARP | Resolves IPv4 address to MAC on a local network |
| NDP | IPv6 neighbour and router discovery |
| DNS | Resolves names to addresses |
| DHCP | Assigns network configuration automatically |
| NAT | Translates IP addresses |
| PAT | Shares an address using port translation |
| Default gateway | Route to destinations outside the local subnet |
| VLAN | Logical Layer 2 broadcast domain |

**Key troubleshooting principle:** If a device cannot reach a destination, verify the local IP configuration, subnet mask, gateway, DNS configuration and route. Test each layer separately rather than changing several settings at once.

# 5. Switching, Routing and Network Infrastructure

## 5.1 Network Devices

| Device | Main function |
|---|---|
| Hub | Repeats incoming signals to all ports; largely obsolete |
| Layer 2 switch | Forwards Ethernet frames using MAC addresses |
| Layer 3 switch | Performs switching and IP routing |
| Router | Connects different IP networks and forwards packets |
| Firewall | Allows or blocks traffic according to security rules |
| Access Point (AP) | Connects wireless clients to a LAN |
| Wireless LAN Controller (WLC) | Centrally manages multiple access points |
| Repeater | Regenerates a signal to extend its reach |

**Layer 2 vs Layer 3 switching:**
- Layer 2 switches forward frames based on destination MAC addresses.
- Layer 3 switches can route packets between IP networks, including VLANs.
- A Layer 2 switch alone cannot route traffic between VLANs.

## 5.2 VLANs — Network Segmentation

A **VLAN (Virtual LAN)** logically separates a switched network into different Layer 2 broadcast domains.

Example:

| VLAN | Purpose | Example subnet |
|---|---|---|
| 10 | Users | 192.168.10.0/24 |
| 20 | Servers | 192.168.20.0/24 |
| 30 | Guest Wi-Fi | 192.168.30.0/24 |
| 99 | Management | 192.168.99.0/24 |

Benefits:
- Separates devices by function or security level.
- Reduces broadcast traffic within each segment.
- Makes network policies easier to manage.
- Helps isolate guest, user, server and management networks.

Devices in different VLANs cannot communicate directly at Layer 2. Communication between them requires Layer 3 routing and may be restricted by ACLs or firewalls.

### Access ports

An access port normally carries traffic for one VLAN. It is typically used for endpoint devices such as PCs, printers and IP phones.

Cisco IOS example:

```text
configure terminal
vlan 10
 name USERS
exit

interface gigabitEthernet 1/0/10
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit
```

Use PortFast only on appropriate endpoint-facing ports, not ordinary switch-to-switch links.

### Trunk ports

A trunk carries traffic for multiple VLANs, usually between switches or between a switch and a router.

IEEE 802.1Q adds VLAN tags to Ethernet frames so the receiving device knows which VLAN they belong to.

```text
interface gigabitEthernet 1/0/48
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
```

Important:
- Access port = normally one VLAN.
- Trunk port = multiple VLANs.
- Both ends must have compatible trunk settings.
- Check the allowed VLAN list on both ends.
- The native VLAN carries untagged traffic on an 802.1Q trunk; its configuration must match on both ends.
- Avoid using the default VLAN for user or management traffic where possible.

## 5.3 How a Switch Forwards Traffic

A switch learns which devices are reachable through its ports by examining the **source MAC address** of incoming frames.

Basic process:
1. A frame arrives on a switch port.
2. The switch reads the source MAC address.
3. It records the MAC address and incoming port in its MAC address table.
4. It checks the destination MAC address.
5. It forwards the frame through the appropriate port if known.

If the destination MAC is unknown, the switch floods the frame within the relevant VLAN, excluding the incoming port. Broadcast frames are also flooded within that VLAN.

Useful command:

```text
show mac address-table
```

### MAC table vs ARP table

| Table | Maps | Typically used by |
|---|---|---|
| MAC address table | MAC address → switch port | Switch |
| ARP table | IPv4 address → MAC address | Host/router |

Do not confuse the two: a switch uses its MAC table to forward Ethernet frames, while ARP helps a host or router find the local MAC address associated with an IPv4 address.

## 5.4 STP — Spanning Tree Protocol

Redundant switch links improve availability, but Layer 2 loops can cause:
- Broadcast storms.
- Duplicate frames.
- MAC address table instability.
- Network outages.

**STP** prevents Layer 2 loops by creating a loop-free logical topology. It blocks redundant paths while keeping them available as backups.

Common versions:
- STP (802.1D) — original version, slower convergence.
- RSTP (802.1w) — faster convergence.
- MSTP (802.1s) — maps multiple VLANs to spanning-tree instances.
- Rapid PVST+ — Cisco implementation with a spanning-tree instance per VLAN.

### Root bridge

STP elects a **root bridge**. The election is based on the lowest Bridge ID, which consists of bridge priority and MAC address. Lower priority is preferred; if priorities are equal, the lower MAC address wins.

Useful commands:

```text
show spanning-tree
show spanning-tree vlan 10
```

Security and configuration:
- Plan which switch should be the root bridge.
- Use PortFast on suitable endpoint-facing ports.
- Enable BPDU Guard on endpoint-facing ports where appropriate.
- Do not disable STP simply to remove a suspected loop; identify and correct the topology problem.

Example Cisco configuration:

```text
spanning-tree mode rapid-pvst

interface gigabitEthernet 1/0/10
 spanning-tree portfast
 spanning-tree bpduguard enable
```

## 5.5 Inter-VLAN Routing

VLANs are separate Layer 2 networks. Devices in different VLANs need a Layer 3 device to communicate.

Two common methods:

### Method 1: Router-on-a-stick

- A router connects to a switch through an 802.1Q trunk.
- The router uses logical subinterfaces.
- Each subinterface is associated with a VLAN and acts as its gateway.
- It is relatively simple, but the router link can become a performance bottleneck.

### Method 2: Layer 3 switch with SVIs

An **SVI (Switched Virtual Interface)** is a virtual Layer 3 interface associated with a VLAN.

Example:

```text
configure terminal
ip routing

interface vlan 10
 description USERS_GATEWAY
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

Hosts in VLAN 10 use `192.168.10.1` as their default gateway.

Useful commands:

```text
show ip interface brief
show vlan brief
show interfaces trunk
show ip route
```

If inter-VLAN communication fails, check:
1. The VLAN exists and is active.
2. The endpoint port belongs to the correct VLAN.
3. The trunk allows the required VLAN.
4. The SVI or router subinterface is up.
5. The host has the correct IP address and gateway.
6. Routing and firewall/ACL policies allow the traffic.

## 5.6 Routing Basics

Routing moves packets between different IP networks.

A router uses a **routing table** to decide where to send packets. The most specific matching route (longest prefix match) is preferred.

### Static routing

A static route is manually configured by an administrator.

Advantages:
- Simple for small or predictable networks.
- No routing protocol overhead.
- Useful for default routes and specific paths.

Disadvantages:
- Must be updated manually.
- Does not automatically adapt to topology changes unless another mechanism is configured.

Cisco example:

```text
ip route 192.168.20.0 255.255.255.0 192.168.1.2
```

This sends traffic for `192.168.20.0/24` to next-hop `192.168.1.2`.

Default route example:

```text
ip route 0.0.0.0 0.0.0.0 192.168.1.1
```

### Dynamic routing

Dynamic routing protocols exchange route information between routers.

| Protocol | Practical use |
|---|---|
| RIP | Older, simple distance-vector protocol; limited scalability |
| OSPF | Common link-state protocol used within an organization |
| BGP | Exchanges routing information between autonomous systems; important for the Internet |

Basic OSPF example:

```text
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
```

Useful commands:

```text
show ip route
show ip protocols
show ip ospf neighbor
```

Do not introduce a routing protocol or change route advertisements without understanding the existing topology and routing policy.

## 5.7 EtherChannel and Link Aggregation

**EtherChannel** combines multiple physical links into one logical link.

Benefits:
- Increases available aggregate bandwidth.
- Provides link redundancy.
- Simplifies management of multiple physical links.

**LACP (Link Aggregation Control Protocol)** negotiates link aggregation between compatible devices.

Cisco example:

```text
interface range gigabitEthernet 1/0/1-2
 channel-group 1 mode active
```

Verification:

```text
show etherchannel summary
```

Important:
- Both sides must have compatible settings.
- Member links should have compatible speed, duplex and VLAN/trunk configuration.
- A single traffic flow may not use the full combined bandwidth; distribution depends on the device's hashing method.

## 5.8 Physical Network Infrastructure

### Copper and fiber

| Medium | Characteristics | Typical use |
|---|---|---|
| UTP | Twisted copper pairs; common and inexpensive | Office Ethernet |
| STP | Shielded twisted-pair copper; can reduce electromagnetic interference when properly installed | Electrically noisy environments |
| Single-mode fiber (SMF) | Smaller core, longer distances | Campus/WAN and long-distance links |
| Multi-mode fiber (MMF) | Larger core, shorter distances | Data centers and building links |

Fiber is resistant to electromagnetic interference. Check that fiber type, connector, transceiver and supported distance match at both ends.

### SFP transceivers

SFP modules connect switches and other network devices to fiber or copper links.

Before replacing a module, verify:
- SFP type and compatibility with the device.
- Fiber type: single-mode or multi-mode.
- Connector type.
- Wavelength and supported distance.
- Speed supported by both ends.

### PoE — Power over Ethernet

PoE supplies electrical power over Ethernet cabling.

| Standard | Common name | Maximum power supplied by port |
|---|---|---:|
| IEEE 802.3af | PoE | 15.4 W |
| IEEE 802.3at | PoE+ | 30 W |
| IEEE 802.3bt Type 3 | PoE++ | 60 W |
| IEEE 802.3bt Type 4 | PoE++ | 90 W |

Common PoE devices:
- Wireless access points.
- IP phones.
- IP cameras.
- Some access-control devices.

Check the switch's total PoE budget as well as the power requirement of each connected device.

## 5.9 Cisco IOS — Essential Verification Commands

| Command | Purpose |
|---|---|
| `show running-config` | Displays active configuration |
| `show startup-config` | Displays saved configuration |
| `show ip interface brief` | Summarizes interface status and IP addresses |
| `show interfaces` | Displays interface details and counters |
| `show vlan brief` | Lists VLANs and assigned access ports |
| `show interfaces trunk` | Shows trunk status and allowed VLANs |
| `show mac address-table` | Displays learned MAC addresses |
| `show spanning-tree` | Displays STP state and topology |
| `show etherchannel summary` | Displays link aggregation status |
| `show ip route` | Displays IPv4 routing table |
| `show arp` | Displays ARP entries |
| `show cdp neighbors detail` | Displays Cisco neighbour information |
| `show lldp neighbors detail` | Displays LLDP neighbour information |

### Safe change procedure

Before changing a switch or router:

1. Check the current configuration and device status.
2. Confirm the correct device, interface and VLAN.
3. Save or back up the current configuration.
4. Understand the expected impact, especially on remote access.
5. Make one controlled change at a time.
6. Verify connectivity and device status.
7. Save the configuration only after successful verification.
8. Keep a rollback plan.

**Remember:** VLANs provide Layer 2 segmentation, trunks carry multiple VLANs, STP prevents Layer 2 loops, and routers or Layer 3 switches provide communication between IP networks.

# 6. Network and System Troubleshooting

## 6.1 General Troubleshooting Method

Use a structured approach instead of changing several settings at once.

1. **Define the problem:** What does not work? One device, one user, one application or the whole network?
2. **Determine the scope:** Compare affected and working devices.
3. **Check recent changes:** Updates, configuration changes, new hardware, VLAN changes or firewall rules.
4. **Collect evidence:** Commands, logs, timestamps, error messages and system state.
5. **Identify the cause:** Test one layer or component at a time.
6. **Apply the smallest safe change:** Avoid unnecessary configuration changes.
7. **Verify:** Confirm that the original problem is resolved and other services still work.
8. **Document:** Record the cause, action, result and rollback procedure.

**Network troubleshooting order:**

`Link → IP → Gateway → Routing → DNS → Port/Service → Firewall → Logs → Application`

## 6.2 Network Troubleshooting — Step by Step

### Step 1: Check the physical link and network adapter

Verify:
- Ethernet cable is connected and undamaged.
- Wi-Fi is enabled and connected to the correct SSID.
- Network adapter is enabled.
- Switch port is active.
- Correct VLAN is assigned.

Linux:

```bash
ip -br link
ip link show
```

Windows PowerShell:

```powershell
Get-NetAdapter
Get-NetAdapter | Format-Table Name, Status, LinkSpeed
```

### Step 2: Check IP configuration

Verify:
- IP address.
- Subnet mask/prefix.
- Default gateway.
- DNS servers.
- DHCP status.
- Duplicate IP address.

Linux:

```bash
ip -br addr
ip route
```

Windows:

```cmd
ipconfig /all
```

If Windows has an address in `169.254.0.0/16`, it may not have received a DHCP lease.

To renew a DHCP lease on Windows:

```cmd
ipconfig /release
ipconfig /renew
```

These commands temporarily disconnect the current IP configuration.

### Step 3: Test the default gateway

The gateway should normally be reachable from the local subnet.

Linux:

```bash
ping -c 4 192.168.1.1
```

Windows:

```cmd
ping 192.168.1.1
```

If the gateway cannot be reached, check:
- IP address and subnet mask.
- VLAN assignment.
- Physical link.
- Gateway interface/SVI status.
- ARP/NDP neighbour resolution.
- Local firewall rules.

A failed ping does not always mean the device is unavailable; ICMP may be blocked.

### Step 4: Check routing

Linux:

```bash
ip route
ip route get 8.8.8.8
```

Windows:

```cmd
route print
```

PowerShell:

```powershell
Get-NetRoute
```

Check that:
- A suitable route exists.
- The default gateway is correct.
- The next-hop address is reachable.
- No unexpected route overrides the intended path.

### Step 5: Test DNS

If access by IP address works but access by hostname fails, investigate DNS.

Linux:

```bash
getent hosts example.com
resolvectl status
dig example.com
```

Windows:

```cmd
nslookup example.com
ipconfig /flushdns
```

PowerShell:

```powershell
Resolve-DnsName example.com
Clear-DnsClientCache
```

Check:
- Configured DNS server.
- DNS server reachability.
- DNS record correctness.
- DNS suffix/search configuration.
- Resolver cache.
- Firewall rules for DNS (UDP/TCP 53).

### Step 6: Test a service port

A host can respond to ping while a specific application port is unavailable.

Linux:

```bash
nc -vz server.example.com 443
ss -lntup
```

Windows PowerShell:

```powershell
Test-NetConnection server.example.com -Port 443
Get-NetTCPConnection -State Listen
```

Check:
- Is the service running?
- Is it listening on the expected address and port?
- Is the correct protocol being used?
- Is a firewall or ACL blocking traffic?
- Is the server reachable through the expected route?

### Step 7: Trace the network path

Linux:

```bash
traceroute 8.8.8.8
tracepath 8.8.8.8
```

Windows:

```cmd
tracert 8.8.8.8
pathping 8.8.8.8
```

These tools help identify where traffic may stop or experience loss. Some routers do not respond to traceroute probes, so a missing hop is not conclusive proof of failure.

### Step 8: Inspect packets when necessary

Packet capture can show whether requests leave the host and whether responses return.

Linux:

```bash
sudo tcpdump -ni any host 10.0.0.10
sudo tcpdump -ni any port 53
sudo tcpdump -ni eth0 -w /tmp/capture.pcap
```

Useful Wireshark display filters:

```text
dns
tcp.port == 443
ip.addr == 10.0.0.10
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

Use packet captures only within your authorized scope. PCAP files can contain credentials, personal data or other sensitive information. Restrict access and retention.

### Step 9: Check logs

Linux:

```bash
journalctl -p warning
journalctl -u ssh
journalctl -b
```

Common log locations:

```text
/var/log/syslog
/var/log/auth.log
/var/log/kern.log
```

Windows:
- Event Viewer → System: drivers, services, hardware and startup.
- Event Viewer → Application: application errors.
- Event Viewer → Security: authentication and audit events.
- Windows Defender Firewall logs: blocked or allowed traffic, if logging is enabled.

PowerShell:

```powershell
Get-WinEvent -LogName System -MaxEvents 50
Get-WinEvent -LogName Application -MaxEvents 50
```

Always correlate log timestamps with the reported incident.

## 6.3 Common Symptoms and Likely Causes

| Symptom | Possible causes / what to check |
|---|---|
| `169.254.x.x` address | DHCP failure, VLAN, relay, exhausted scope or physical link |
| IP works, hostname does not | DNS resolver, record, suffix or cache |
| Gateway works, Internet does not | Upstream routing, NAT, firewall or ISP |
| DNS works, TCP port fails | Service not listening, firewall, ACL, routing or NAT |
| Only one application fails | Application, port, TLS, proxy or application-specific firewall rule |
| One-way communication | Asymmetric routing, stateful firewall, ACL or incorrect gateway |
| Slow connection / packet loss | Interface errors, speed/duplex mismatch, Wi-Fi interference, congestion or MTU |
| Only one VLAN fails | Access VLAN, trunk allowed/native VLAN, SVI, DHCP relay or ACL |
| Large transfers fail | MTU/MSS, Path MTU Discovery or tunnel configuration |
| IP address conflict | Duplicate static IP, DHCP conflict or stale ARP entry |
| TLS/certificate error | Incorrect system time, expired certificate, wrong hostname/SAN, trust chain or proxy inspection |

## 6.4 System Troubleshooting

### High CPU usage

Linux:

```bash
top
ps aux --sort=-%cpu | head
```

Windows PowerShell:

```powershell
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
```

Check:
- Which process consumes CPU?
- Is the usage temporary or continuous?
- Did a service or scheduled task recently change?
- Are there unusual processes or unexpected child processes?

Do not terminate an unknown process before identifying its purpose.

### High memory usage

Linux:

```bash
free -h
top
```

Windows:

```powershell
Get-Process | Sort-Object WorkingSet64 -Descending |
    Select-Object -First 10
```

Check for memory pressure, excessive processes, leaks and unusual application behaviour.

### Disk space problems

Linux:

```bash
df -h
du -sh /var/log/*
lsblk
```

Windows PowerShell:

```powershell
Get-Volume
Get-PSDrive -PSProvider FileSystem
```

Check:
- Free disk space.
- Large log files.
- Temporary files.
- Application data growth.
- Disk health and filesystem errors.

Do not delete logs or application data blindly. Confirm retention requirements and identify what is consuming space first.

### Service problems

Linux:

```bash
systemctl status <service>
journalctl -u <service> --since today
```

Windows PowerShell:

```powershell
Get-Service
Get-Service -Name <ServiceName>
```

Check service status, startup configuration, dependencies, permissions and related logs before restarting it.

## 6.5 Safe Troubleshooting Checklist

Before making changes:

- [ ] Confirm the affected device, user, service and time.
- [ ] Check whether other devices have the same issue.
- [ ] Record current IP, route and adapter configuration.
- [ ] Check recent changes and relevant logs.
- [ ] Save or back up configuration before editing it.
- [ ] Confirm that you have authorization.
- [ ] Change one thing at a time.
- [ ] Verify the result.
- [ ] Document the fix and rollback method.

**Golden rule:** Diagnose first, collect evidence, then make the smallest reversible change. Avoid rebooting, disabling firewalls, flushing configurations or terminating processes as a first response.

# 7. Cybersecurity Fundamentals and System Hardening

## 7.1 Core Security Principles

| Principle | Practical meaning |
|---|---|
| Confidentiality | Only authorized people can access information |
| Integrity | Data and systems are protected against unauthorized changes |
| Availability | Systems and data remain accessible when needed |
| Least privilege | Users and services receive only necessary permissions |
| Defense in depth | Use multiple independent security controls |
| Zero Trust | Verify access explicitly; do not trust a device or user solely because of network location |

### Common threats

- **Malware:** malicious software, including ransomware, spyware and trojans.
- **Phishing:** attempts to steal credentials or deliver malware through deceptive messages.
- **Credential attacks:** password spraying, brute force and credential theft.
- **Privilege escalation:** obtaining permissions beyond those initially granted.
- **Lateral movement:** moving from one compromised system to another.
- **Denial of Service (DoS):** disrupting service availability.
- **Misconfiguration:** weak permissions, exposed services, default credentials or insecure settings.

## 7.2 Identity and Access Management

Follow these practices:
- Use individual accounts; avoid shared administrator accounts.
- Use separate standard and administrative accounts.
- Enable MFA, especially for privileged and remote access.
- Apply least privilege.
- Remove or disable inactive accounts.
- Review administrator and privileged group membership.
- Use strong, unique passwords and a password manager.
- Never store passwords or secrets in scripts or plain-text files.

Windows — review local users and administrators:

```powershell
Get-LocalUser
Get-LocalGroupMember -Group "Administrators"
```

Linux — review users and groups:

```bash
getent passwd
getent group sudo
id <USER>
```

## 7.3 Linux Hardening

### Updates

```bash
sudo apt update
sudo apt upgrade
```

Install security updates regularly and verify services after maintenance.

### SSH security

Prefer SSH keys over password authentication where practical.

Configuration:

```bash
sudo nano /etc/ssh/sshd_config
```

Common settings to review:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
```

Do not disable password authentication until key-based access has been tested and recovery access is available.

Validate and reload:

```bash
sudo sshd -t
sudo systemctl reload ssh
```

### Permissions

- Avoid `chmod 777`.
- Use appropriate file ownership and group permissions.
- Restrict access to private keys and secret files.
- Review sudo permissions using `visudo`.
- Audit unexpected SUID/SGID files.

Useful commands:

```bash
ls -l
getfacl <FILE>
sudo visudo
find / -type f -perm /6000 2>/dev/null
```

### Firewall

Check existing rules before making changes:

```bash
sudo ufw status verbose
sudo nft list ruleset
```

Example: allow SSH only from a trusted subnet:

```bash
sudo ufw allow from 10.10.0.0/24 to any port 22 proto tcp
```

Be careful when enabling or changing a firewall over SSH. An incorrect rule can lock you out.

## 7.4 Windows Hardening

Important measures:
- Keep Windows and applications patched.
- Enable Microsoft Defender Antivirus and real-time protection.
- Enable Windows Defender Firewall for all network profiles.
- Use standard accounts for daily work.
- Enable MFA for supported accounts and remote access.
- Use BitLocker where supported and protect recovery keys.
- Disable unnecessary services and features.
- Restrict RDP and administrative shares to trusted networks.
- Use application control and attack surface reduction policies where appropriate.
- Enable appropriate security auditing.

### Check Defender status

```powershell
Get-MpComputerStatus
Get-MpThreatDetection
```

### Check firewall profiles

```powershell
Get-NetFirewallProfile |
    Format-Table Name, Enabled, DefaultInboundAction
```

### Check enabled firewall rules

```powershell
Get-NetFirewallRule -Enabled True |
    Select-Object DisplayName, Direction, Action
```

Do not disable Defender or the firewall to troubleshoot an issue without authorization and a controlled alternative protection plan.

## 7.5 Patching and Vulnerability Management

A practical patching process:

1. Identify affected systems and installed software.
2. Review security advisories and vulnerability severity.
3. Prioritize actively exploited and critical vulnerabilities.
4. Test updates where possible.
5. Back up important systems and configurations.
6. Deploy updates in a controlled maintenance window.
7. Restart if required.
8. Verify system and application health.
9. Document exceptions and unresolved vulnerabilities.

A vulnerability scan identifies potential weaknesses; it does not prove that a system is exploitable.

## 7.6 Logging, Integrity and Backups

### Logging

- Centralize important logs.
- Synchronize system clocks using NTP.
- Define log retention periods.
- Restrict access to logs.
- Protect logs from unauthorized modification.
- Monitor authentication, privilege changes and security events.

### File integrity

Linux:

```bash
sha256sum file
```

Windows:

```powershell
Get-FileHash file -Algorithm SHA256
```

A hash helps compare file integrity. A matching hash does not by itself prove that a file is trustworthy.

### Backup — 3-2-1 rule

- Keep at least **3 copies** of important data.
- Use **2 different types of storage**.
- Keep **1 copy off-site**.

Also:
- Encrypt backups.
- Keep offline or immutable copies where possible.
- Restrict backup administration privileges.
- Test restoration regularly.
- Protect backup systems from ransomware and compromised administrator accounts.

## 7.7 System Hardening Checklist

- [ ] Remove unnecessary software and services.
- [ ] Apply current security patches.
- [ ] Use MFA for privileged and remote access.
- [ ] Remove inactive accounts and unnecessary privileges.
- [ ] Enable host firewall and endpoint protection.
- [ ] Restrict remote administration.
- [ ] Secure SSH, RDP and administrative interfaces.
- [ ] Protect secrets, keys and credentials.
- [ ] Centralize logs and synchronize time.
- [ ] Maintain and test secure backups.
- [ ] Document configuration changes and exceptions.

**Remember:** Hardening reduces attack surface. It does not eliminate risk; systems still require monitoring, patching, backups and incident response.

# 8. Network Security

## 8.1 Firewalls and Traffic Filtering

A firewall controls network traffic according to defined rules.

A rule commonly considers:
- Source IP address.
- Destination IP address.
- Protocol (TCP, UDP, ICMP).
- Source and destination port.
- Traffic direction.
- Connection state.
- Action: allow, deny or reject.
- Logging requirements.

### Firewall types

| Type | Function |
|---|---|
| Stateless firewall / ACL | Filters packets using individual packet attributes |
| Stateful firewall | Tracks connection state and related traffic |
| Next-Generation Firewall (NGFW) | May inspect applications, users and threats in addition to IPs and ports |
| Host firewall | Protects an individual endpoint or server |

**Default-deny principle:** Allow only required traffic and restrict everything else according to policy.

### Linux firewall checks

```bash
sudo ufw status verbose
sudo nft list ruleset
```

### Windows firewall checks

```powershell
Get-NetFirewallProfile
Get-NetFirewallRule -Enabled True
```

Do not disable a firewall as a general troubleshooting solution. Identify the specific blocked traffic and create a narrowly scoped rule if authorized.

## 8.2 Network Segmentation

Separate networks according to their purpose and risk.

Example:

| Network | Purpose | Recommended access |
|---|---|---|
| User | Employee workstations | Required business services |
| Server | Application and infrastructure servers | Explicitly permitted services |
| Management | Switches, routers and hypervisors | Administrators only |
| Guest | Visitors and personal devices | Internet only |
| Backup | Backup infrastructure | Restricted backup traffic |
| IoT | Cameras, printers and other devices | Only required services |

Controls:
- Use VLANs to separate Layer 2 broadcast domains.
- Use ACLs and firewalls to control inter-network traffic.
- Restrict management interfaces to trusted networks.
- Avoid unnecessary access between user and server networks.
- Monitor traffic between security zones.

VLANs provide segmentation, but they are not a substitute for firewall rules or access control.

## 8.3 Secure Remote Administration

### SSH

- Prefer key-based authentication.
- Disable direct root login where appropriate.
- Restrict access to trusted source networks.
- Keep the SSH server updated.
- Monitor authentication logs.

### RDP

- Do not expose RDP directly to the Internet.
- Use VPN or a controlled remote-access gateway.
- Require MFA where supported.
- Restrict access to authorized users and systems.
- Monitor successful and failed logons.

### VPN

A VPN provides an encrypted connection over an untrusted network.

Common uses:
- Remote employee access.
- Site-to-site connectivity.
- Secure administration.

Security requirements:
- Strong authentication and MFA.
- Updated VPN gateways and clients.
- Restricted access after connection.
- Logging and monitoring.
- Removal of access when no longer required.

## 8.4 Common Network Security Controls

| Control | Purpose |
|---|---|
| ACL | Restricts traffic based on defined conditions |
| IDS | Detects suspicious network activity |
| IPS | Detects and can block suspicious activity |
| DHCP Snooping | Helps prevent unauthorized DHCP servers |
| Dynamic ARP Inspection | Helps protect against certain ARP spoofing attacks |
| Port Security | Limits or controls MAC addresses on switch ports |
| 802.1X | Provides port-based network access authentication |
| SNMPv3 | Provides authenticated and optionally encrypted network management |
| DNS filtering | Blocks or alerts on access to selected domains |
| Egress filtering | Restricts outbound network connections |

Enable controls according to the capabilities of the infrastructure and the organization's security policy.

## 8.5 Network Security Checklist

- [ ] Separate user, server, guest and management networks.
- [ ] Restrict inter-VLAN traffic with ACLs or firewalls.
- [ ] Disable unused switch ports.
- [ ] Use secure management protocols.
- [ ] Restrict SSH, RDP and device management access.
- [ ] Use MFA for remote access.
- [ ] Prefer SNMPv3.
- [ ] Monitor firewall and network device logs.
- [ ] Review firewall rules periodically.
- [ ] Keep network device firmware updated.
- [ ] Maintain backups of device configurations.

**Remember:** A secure network is not simply one with a firewall. It requires segmentation, restricted access, secure administration, monitoring and regular review.

# 9. SOC, Security Monitoring and Incident Response

## 9.1 SOC Fundamentals

A **Security Operations Center (SOC)** monitors systems and networks, investigates security alerts and coordinates incident response.

Typical responsibilities:
- Monitor SIEM, EDR and security alerts.
- Investigate suspicious activity.
- Identify affected users, devices and systems.
- Correlate endpoint, network, identity and cloud events.
- Escalate confirmed or high-risk incidents.
- Document evidence and actions.

### Important terms

| Term | Meaning |
|---|---|
| SIEM | Collects and correlates logs and security events |
| EDR | Endpoint Detection and Response; monitors and responds to endpoint threats |
| IOC | Indicator of Compromise, such as a suspicious IP, domain or file hash |
| IOA | Indicator of Attack; suspicious behaviour or activity |
| TTP | Tactics, Techniques and Procedures used by an attacker |
| False positive | Alert that does not represent malicious activity |
| False negative | Malicious activity that was not detected |

An IOC can become outdated or be changed by an attacker. Behavioural evidence and context are often more useful than a single indicator.

## 9.2 Alert Triage

Use a consistent process when investigating an alert.

### Step 1: Understand the alert

Record:
- Alert name and detection rule.
- Severity and timestamp.
- Hostname and IP address.
- User account.
- Source of telemetry.
- Reason for detection.

### Step 2: Investigate the process

Collect:
- Process name and full path.
- Command line.
- Process ID (PID).
- Parent and child processes.
- Start time.
- User account and integrity level.
- Digital signature.
- File hash.

### Step 3: Check context

Ask:
- Is this host a workstation or server?
- Is the activity expected for this user's role?
- Was there an approved change or maintenance window?
- Is the same behaviour present on other hosts?
- Is the activity associated with known administrative software?

### Step 4: Look for persistence

Check for:
- New or modified services.
- Scheduled tasks.
- Registry Run keys and startup entries.
- New accounts or group membership changes.
- Unexpected scripts or executables.
- Suspicious WMI activity.

### Step 5: Check network activity

Review:
- DNS queries.
- Destination IPs and ports.
- Domains and URLs.
- Process owning the connection.
- Connection frequency and timing.
- Proxy or VPN context.

### Step 6: Classify and document

Possible conclusions:
- Benign.
- Suspicious.
- Malicious.
- Inconclusive.

Record the evidence supporting the conclusion. Do not classify an event based only on a process name, IP address or file hash.

## 9.3 Important Windows Event IDs

| Log / Event ID | Meaning | What to investigate |
|---|---|---|
| Security 4624 | Successful logon | Account, logon type, source and time |
| Security 4625 | Failed logon | Repeated failures, account and source |
| Security 4648 | Logon using explicit credentials | Process, account and destination |
| Security 4672 | Special privileges assigned | Whether privileged access was expected |
| Security 4688 | Process creation | Command line, parent process and user |
| Security 4697 / System 7045 | Service installed | Service path, account and initiator |
| Security 4698 | Scheduled task created | Task action, author and trigger |
| Security 4720 | User account created | Who created it and why |
| Security 4728 | Member added to global security group | Group and initiator |
| Security 4732 | Member added to local security group | Group and initiator |
| PowerShell 4103 | Module logging | Commands and modules, if enabled |
| PowerShell 4104 | Script block logging | Script content, if enabled |

Event IDs depend on audit policy, operating system version and logging configuration. The absence of an event does not prove that an action did not happen.

### Quick PowerShell event searches

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624,4625,4688
    StartTime = (Get-Date).AddHours(-4)
} | Select-Object TimeCreated, Id, Message
```

PowerShell operational log:

```powershell
Get-WinEvent -LogName 'Microsoft-Windows-PowerShell/Operational' -MaxEvents 100
```

## 9.4 Sysmon and Investigation Tools

**Sysmon** provides additional Windows telemetry. It must be installed and configured; available events depend on the configuration.

| Sysmon Event ID | Meaning |
|---|---|
| 1 | Process creation |
| 3 | Network connection |
| 7 | Image loaded |
| 10 | Process access |
| 11 | File created |
| 12–14 | Registry events |
| 22 | DNS query |

Useful tools:

| Tool | Purpose |
|---|---|
| Task Manager / Process Explorer | Process tree, handles, DLLs and process details |
| Autoruns | Reviews startup and persistence locations |
| TCPView | Shows processes and network connections |
| Sysmon | Additional endpoint telemetry |
| Microsoft Defender / EDR | Threat detection, investigation and response |
| Wireshark / tcpdump | Packet analysis |
| SIEM | Centralized event search and correlation |

A signed or familiar executable is not automatically safe. Investigate its path, command line, parent process, hash, signature and behaviour.

## 9.5 MITRE ATT&CK

MITRE ATT&CK is a knowledge base of adversary tactics and techniques.

Examples:

| Technique | ID | Description |
|---|---|---|
| Command and Scripting Interpreter | T1059 | Use of command shells or scripting languages |
| Scheduled Task | T1053.005 | Persistence or execution through scheduled tasks |
| Windows Service | T1543.003 | Creation or modification of a Windows service |
| Registry Run Keys / Startup | T1547.001 | Persistence through startup locations |
| OS Credential Dumping | T1003 | Attempts to obtain credentials |
| Remote Services | T1021 | Use of remote services for lateral movement |
| Impair Defenses | T1562.001 | Disabling or weakening security tools |

Use ATT&CK to describe observed behaviour and support investigation. A technique mapping alone is not proof of malicious activity.

## 9.6 Incident Response

Follow the organization's incident response plan.

1. **Identify:** Confirm the alert and determine the affected systems and accounts.
2. **Preserve evidence:** Record timestamps, logs, hashes, command lines, process details and network connections.
3. **Assess impact:** Determine the criticality of affected hosts, accounts and services.
4. **Contain:** Apply approved actions, such as endpoint isolation, blocking an IOC or revoking sessions.
5. **Escalate:** Notify the incident response team and relevant stakeholders.
6. **Eradicate and recover:** Remove the cause, restore systems safely and verify their integrity.
7. **Monitor:** Check for persistence, lateral movement and recurring activity.
8. **Document:** Record decisions, actions, evidence and lessons learned.

### Do not

- Delete suspicious files before collecting evidence, unless immediate containment procedures require it.
- Clear event logs.
- Disable EDR or antivirus to make an alert disappear.
- Run suspected malware.
- Reset passwords without considering active sessions, tokens and the incident response procedure.
- Isolate critical production systems without considering business impact and authorization.

**Remember:** Observe → Preserve context → Verify → Assess impact → Escalate or take approved action.

# 10. Essential Administration Procedures and Reference

## 10.1 Before Making a Change

Use this checklist for system, network and security changes.

- [ ] Confirm the request and authorization.
- [ ] Identify affected systems, users and services.
- [ ] Check dependencies and expected impact.
- [ ] Record the current configuration.
- [ ] Back up relevant data or configuration.
- [ ] Prepare a rollback procedure.
- [ ] Schedule a maintenance window if required.
- [ ] Notify affected users or teams.

## 10.2 After Making a Change

- [ ] Verify the configuration.
- [ ] Test the affected service.
- [ ] Confirm dependent services still work.
- [ ] Check logs and monitoring.
- [ ] Confirm that security controls remain enabled.
- [ ] Save the configuration if appropriate.
- [ ] Document the result and any outstanding issues.

## 10.3 Useful Administration Commands

| Task | Linux | Windows |
|---|---|---|
| Current user | `whoami` | `whoami` |
| Hostname | `hostname` | `hostname` |
| IP configuration | `ip addr` | `ipconfig /all` |
| Routing table | `ip route` | `route print` |
| DNS lookup | `dig example.com` | `nslookup example.com` |
| Connectivity test | `ping <HOST>` | `ping <HOST>` |
| Trace route | `traceroute <HOST>` | `tracert <HOST>` |
| Listening ports | `ss -lntup` | `netstat -ano` |
| Processes | `ps aux` | `Get-Process` |
| Services | `systemctl --type=service` | `Get-Service` |
| System logs | `journalctl` | Event Viewer / `Get-WinEvent` |
| Disk usage | `df -h` | `Get-Volume` |
| File hash | `sha256sum <FILE>` | `Get-FileHash <FILE> -Algorithm SHA256` |

## 10.4 Incident and Change Documentation

For every significant change or incident, record:

| Field | Information |
|---|---|
| Date/time | Include timezone |
| System | Hostname, IP and environment |
| Request/incident | Ticket or incident reference |
| Description | What happened or what was changed |
| Evidence | Logs, commands, screenshots or hashes |
| Action | What was done and by whom |
| Result | Whether the issue was resolved |
| Rollback | How to restore the previous state |
| Follow-up | Remaining risks or required actions |

## 10.5 Final Operational Rules

1. Diagnose before changing configuration.
2. Use least privilege.
3. Never expose management interfaces unnecessarily.
4. Keep systems patched and monitored.
5. Back up configurations and important data.
6. Test restoration, not just backup creation.
7. Preserve evidence during security incidents.
8. Document changes and verify their results.
9. Follow organizational procedures and escalation paths.
10. Never run unknown scripts or commands with elevated privileges without understanding their effects.

**Core principle:** Make controlled, authorized and reversible changes. Verify the result and leave a clear record for the next administrator.
