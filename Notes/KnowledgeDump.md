# IT compressed notes
> **Purpose:** A compact, practical reference for common administration, troubleshooting and security tasks.

# 1. Linux Administration

**Scope:** Debian-based systems. Package names, service names and paths may differ on other distributions.

## 1.1 Filesystem essentials

| Path | Purpose | Path | Purpose |
|---|---|---|---|
| `/etc` | System configuration | `/var/log` | System/application logs |
| `/home` | User home directories | `/root` | Root user's home |
| `/var` | Variable data, logs, caches | `/tmp` | Temporary files |
| `/usr` | Programs and libraries | `/opt` | Optional/third-party software |
| `/boot` | Kernel and boot files | `/dev` | Device files |
| `/proc` | Process/kernel information | `/sys` | Kernel/hardware information |

Important files: `/etc/passwd` (accounts), `/etc/group` (groups), `/etc/fstab` (automatic mounts), `/etc/ssh/sshd_config` (SSH server configuration).

## 1.2 Files and text

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `pwd` | Show current directory | `ls -lah` | List files, including hidden files and readable sizes |
| `cd /etc` | Change directory | `cd ..` / `cd ~` | Parent / home directory |
| `find /var/log -name "*.log"` | Find matching files | `file <FILE>` | Identify file type |
| `stat <FILE>` | Show file metadata | `command -v <CMD>` | Locate a command |
| `cp <SRC> <DEST>` | Copy file | `cp -a <DIR> <DEST>` | Copy directory preserving attributes |
| `mv <OLD> <NEW>` | Move or rename | `mkdir -p <PATH>` | Create directory tree |
| `rm -i <FILE>` | Delete with confirmation | `tar -czf backup.tgz <DIR>` | Create compressed archive |
| `tar -xzf backup.tgz` | Extract archive | `less <FILE>` | Read a file interactively |
| `head -n 20 <FILE>` | First 20 lines | `tail -n 50 <FILE>` | Last 50 lines |
| `tail -f <LOG>` | Follow a live log | `grep -n "error" <FILE>` | Search text with line numbers |
| `grep -Rni "failed" /var/log` | Recursive, case-insensitive search | `sort <FILE> \| uniq -c` | Sort and count repeated lines |
| `cut -d: -f1 /etc/passwd` | Extract usernames | `awk -F: '{print $1}' <FILE>` | Print first colon-separated field |

**Pipes/redirection:** `cmd1 | cmd2` passes output to another command; `>` overwrites a file; `>>` appends; `2>` redirects errors. Check the destination before using `>` because it truncates existing content. `rm -rf` is destructive—verify the path, especially with `sudo`.

## 1.3 Users, groups and permissions

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `whoami` | Current username | `id` | UID, GID and group membership |
| `groups` | Groups for current user | `sudo -l` | Commands current user may run with sudo |
| `sudo adduser <USER>` | Create a Debian user | `sudo passwd <USER>` | Set/change password |
| `sudo usermod -aG sudo <USER>` | Add user to sudo group | `sudo passwd -l <USER>` | Lock password authentication |
| `getent passwd <USER>` | Look up account | `getent group <GROUP>` | Look up group |
| `chmod 640 <FILE>` | Set numeric permissions | `chmod u+x <FILE>` | Add execute permission for owner |
| `chown user:group <FILE>` | Change owner and group | `chgrp <GROUP> <FILE>` | Change group ownership |
| `getfacl <FILE>` | View ACLs | `setfacl -m u:<USER>:r <FILE>` | Grant user read ACL |

`usermod -aG` must include `-a`; without it, existing supplementary groups can be replaced.

Permission values: `r=4` read, `w=2` write, `x=1` execute.

| Mode | Meaning | Mode | Meaning |
|---|---|---|---|
| `755` | Owner full; others read/execute | `750` | Owner full; group read/execute |
| `700` | Owner only | `644` | Owner read/write; others read |
| `640` | Owner read/write; group read | `600` | Owner read/write only |

Avoid `chmod 777`. Use the minimum permissions needed. `umask 027` removes group write and all permissions for others from newly created objects, subject to the application's requested mode.

## 1.4 Packages, processes and services

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `sudo apt update` | Refresh repository indexes | `sudo apt upgrade` | Upgrade installed packages |
| `sudo apt install <PKG>` | Install package | `sudo apt remove <PKG>` | Remove package, usually retaining config |
| `sudo apt purge <PKG>` | Remove package and system config | `sudo apt autoremove` | Remove unused dependencies |
| `apt search <NAME>` | Search packages | `apt show <PKG>` | Show package details |
| `dpkg -l` | List installed packages | `dpkg -S <PATH>` | Find package owning a file |
| `ps aux` | List processes | `pgrep -af <NAME>` | Find process by name/pattern |
| `top` | Live process/resource view | `free -h` | RAM and swap usage |
| `uptime` | Uptime and load average | `kill -TERM <PID>` | Request graceful termination |
| `kill -KILL <PID>` | Force termination; last resort | `systemctl status <SVC>` | Service state and recent logs |
| `sudo systemctl start <SVC>` | Start service | `sudo systemctl stop <SVC>` | Stop service |
| `sudo systemctl restart <SVC>` | Restart service | `sudo systemctl reload <SVC>` | Reload config if supported |
| `sudo systemctl enable <SVC>` | Start at boot | `sudo systemctl enable --now <SVC>` | Enable and start now |
| `systemctl --failed` | List failed units | `systemctl list-units` | List loaded units |

`apt update` only refreshes package indexes; it does not install updates. Prefer `TERM` before `KILL`. Confirm a service's purpose and dependencies before stopping or disabling it.

## 1.5 Logs and boot

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `journalctl -b` | Current boot's journal | `journalctl -p err -b` | Errors from current boot |
| `journalctl -u <SVC>` | Logs for service | `journalctl -u <SVC> -f` | Follow service log live |
| `journalctl --since today` | Logs since today | `journalctl --since "1 hour ago"` | Recent logs |
| `dmesg -T` | Kernel messages with timestamps | `systemctl --failed` | Failed services/units |
| `systemd-analyze blame` | Boot-time contribution by unit | `cat /proc/cmdline` | Kernel boot parameters |

Common files, if configured: `/var/log/syslog` (general), `/var/log/auth.log` (authentication), `/var/log/kern.log` (kernel). Many systems rely mainly on the systemd journal.

## 1.6 Storage and filesystems

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `lsblk -f` | Disks, partitions and filesystems | `blkid` | Filesystem UUIDs and types |
| `sudo fdisk -l` | List partition tables | `df -h` | Filesystem capacity/free space |
| `du -sh <PATH>` | Directory size | `findmnt` | Mounted filesystems |
| `sudo mount <DEV> <DIR>` | Mount filesystem | `sudo umount <DIR>` | Unmount filesystem |
| `sudo pvs` | Show LVM physical volumes | `sudo vgs` | Show volume groups |
| `sudo lvs` | Show logical volumes | `sudo lsof +L1` | Find deleted files still held open |

Example safe mount check:

```bash
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
findmnt /mnt/data
sudo umount /mnt/data
```

For `/etc/fstab`, prefer UUIDs over device names. After editing, run `sudo findmnt --verify` and `sudo mount -a` before rebooting. A bad entry can cause boot or mount problems.

LVM structure: **PV** (physical volume) → **VG** (volume group) → **LV** (logical volume). Back up before resizing or removing storage.

If disk is full, start with `df -h`, then `du -xhd1 /` to find large directories. Do not delete logs or application data blindly.

## 1.7 Networking, SSH and firewall

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `ip -br addr` | Brief IP/interface view | `ip link` | Interface state |
| `ip route` | Routing table | `ip route get <IP>` | Route selected for destination |
| `ip neigh` | ARP/NDP neighbour cache | `ping -c 4 <IP>` | Test ICMP reachability |
| `tracepath <HOST>` | Trace network path | `getent hosts <NAME>` | Resolve using system configuration |
| `resolvectl status` | DNS configuration | `ss -lntup` | Listening TCP/UDP sockets |
| `ss -s` | Socket statistics | `curl -I <URL>` | HTTP response headers |
| `ssh user@host` | Remote SSH login | `scp <FILE> user@host:/PATH` | Copy file over SSH |
| `sudo sshd -t` | Validate SSH config | `sudo systemctl reload ssh` | Reload SSH service |
| `sudo ufw status verbose` | Show UFW rules | `sudo nft list ruleset` | Show nftables rules |

Example UFW rule: `sudo ufw allow from 10.10.0.0/24 to any port 22 proto tcp`.

Before changing SSH/firewall remotely, allow the management path first, validate the config, keep the current session open and test a second session. Packet captures (`tcpdump`) can contain sensitive data; capture only authorised traffic.

**Linux hardening:** patch packages; remove unused accounts/services; restrict sudo; prefer SSH keys; disable direct root SSH login where appropriate; avoid world-writable files; enable firewall; monitor auth logs; maintain and test backups.

## 1.8 Linux quick troubleshooting

| Symptom | First checks | Symptom | First checks |
|---|---|---|---|
| Slow server | `uptime`, `top`, `free -h` | Disk full | `df -h`, `du -xhd1 /`, `lsof +L1` |
| Service unavailable | `systemctl status`, `journalctl -u` | Interface down | `ip -br addr`, `ip link` |
| No connectivity | `ip route`, ping gateway | IP works, name fails | `resolvectl`, `getent hosts` |
| Port unavailable | `ss -lntup`, firewall, remote test | Boot problem | `journalctl -b`, `dmesg -T` |
| SSH fails | Service status, auth log, config, firewall |  |  |

---

# 2. Windows Administration

**Scope:** Windows 10/11 and Windows Server. Distinguish CMD from PowerShell. Domain policy or Intune may override local settings.

## 2.1 Essential tools

Open these with `Win + R` (use Ctrl + Shift + Enter to run as administrator).

| Command | Purpose | Command | Purpose |
|---|---|---|---|
| `taskmgr` | Processes/performance | `services.msc` | Services |
| `eventvwr.msc` | Event Viewer | `devmgmt.msc` | Device Manager |
| `diskmgmt.msc` | Disk Management | `ncpa.cpl` | Network adapters |
| `wf.msc` | Advanced Firewall | `firewall.cpl` | Basic Firewall |
| `sysdm.cpl` | System/domain settings | `msinfo32` | System information |
| `resmon` | Resource Monitor | `perfmon` | Performance Monitor |
| `perfmon /rel` | Reliability Monitor | `taskschd.msc` | Scheduled Tasks |
| `compmgmt.msc` | Computer Management | `mstsc` | Remote Desktop |
| `gpedit.msc` | Local Group Policy* | `secpol.msc` | Local Security Policy* |

`*` Availability depends on Windows edition.

## 2.2 CMD and PowerShell

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `whoami /all` | Current user, groups and privileges | `hostname` | Computer name |
| `systeminfo` | OS, hardware and hotfix details | `ver` | Windows version |
| `tasklist` | Running processes | `tasklist /svc` | Processes and associated services |
| `driverquery /v` | Driver information | `net user` | Local users |
| `net localgroup` | Local groups | `net localgroup Administrators` | Local admin members |
| `Get-ComputerInfo` | System information | `Get-HotFix` | Installed hotfixes |
| `Get-Service` | Services | `Get-Process` | Processes |
| `Get-LocalUser` | Local accounts | `Get-LocalGroupMember -Group Administrators` | Members of local Administrators |
| `Get-Disk` / `Get-Partition` | Disk/partition inventory | `Get-Volume` | Volumes and free space |
| `Get-Command <NAME>` | Find command | `Get-Help <CMD> -Examples` | Command examples |

PowerShell pipeline pattern: `Get-Service | Where-Object Status -ne 'Running'`. `Where-Object` filters, `Select-Object` chooses fields and `Sort-Object` sorts results.

Terminate only after verifying the process:

```cmd
taskkill /PID <PID> /T
```

`/F` forces termination; it may cause data loss or interrupt a service.

## 2.3 Local users and NTFS permissions

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `Get-LocalUser` | List local users | `Get-LocalGroup` | List local groups |
| `Get-LocalGroupMember -Group Administrators` | Review local admins | `Disable-LocalUser -Name <USER>` | Disable account |
| `Enable-LocalUser -Name <USER>` | Enable account | `Add-LocalGroupMember -Group Administrators -Member <USER>` | Add local admin membership |
| `icacls C:\Data` | Inspect NTFS ACL | `icacls C:\Data /grant user:(R)` | Grant read |
| `icacls C:\Data /grant user:(M)` | Grant modify | `icacls C:\Data /remove user` | Remove explicit entry |

NTFS rights: **Full Control** (including permission changes/ownership), **Modify** (read/write/delete), **Read & Execute**, **Read**, **Write**. Inherited flags: `(OI)` files, `(CI)` directories; `/T` applies recursively.

Share permissions and NTFS permissions both affect network-share access. Avoid broad `Everyone: Full Control` permissions. Use standard accounts for daily work and restrict local Administrators membership.

## 2.4 Processes, services and scheduled tasks

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `Get-Process` | List processes | `Get-Process <NAME>` | Find process |
| `Stop-Process -Name <NAME>` | Stop process | `taskkill /IM <NAME>.exe` | Stop by image name |
| `Get-Service` | List services | `Get-Service <NAME>` | Inspect service |
| `Start-Service <NAME>` | Start service | `Restart-Service <NAME>` | Restart service |
| `Stop-Service <NAME>` | Stop service | `sc qc <NAME>` | Inspect service config |
| `Get-ScheduledTask` | List scheduled tasks | `schtasks /query /fo LIST /v` | Detailed task inventory |

Before stopping a service, inspect its dependencies, startup type and service account. For scheduled tasks, check trigger, action, execution account and last result.

## 2.5 Event logs

Open `eventvwr.msc`. Main logs: **System** (drivers/services/boot), **Application** (application errors), **Security** (logons/account activity), **Setup** (installation/updates).

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `Get-WinEvent -LogName System -MaxEvents 50` | Recent System events | `Get-WinEvent -LogName Application -MaxEvents 50` | Recent Application events |
| `Get-WinEvent -FilterHashtable @{LogName='Security';Id=4624,4625} -MaxEvents 30` | Recent successful/failed logons | `perfmon /rel` | Reliability history |

Important Security Event IDs:

| ID | Meaning | ID | Meaning |
|---:|---|---:|---|
| 4624 | Successful logon | 4625 | Failed logon |
| 4648 | Explicit credentials used | 4672 | Special privileges assigned |
| 4688 | Process created (if audited) | 4697 / 7045 | Service installed |
| 4698 | Scheduled task created | 4720 | User account created |
| 4728 | Added to global security group | 4732 | Added to local security group |

Events depend on audit configuration and log retention. Missing events do not prove an action did not happen.

## 2.6 Storage, networking and repair

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `Get-Disk` | Physical disks | `Get-Partition` | Partitions |
| `Get-Volume` | Filesystems and free space | `diskmgmt.msc` | GUI disk management |
| `ipconfig /all` | Full IP configuration | `ipconfig /flushdns` | Clear DNS cache |
| `ipconfig /release` / `/renew` | Release/renew DHCP IPv4 lease | `arp -a` | ARP cache |
| `ping <HOST>` | ICMP reachability | `tracert <HOST>` | Trace route |
| `pathping <HOST>` | Route and loss diagnostics | `nslookup <NAME>` | DNS query |
| `netstat -ano` | Connections/listeners and PIDs | `route print` | Routing table |
| `Get-NetAdapter` | Adapter state | `Get-NetIPConfiguration` | IP, gateway and DNS |
| `Get-NetRoute` | Routing table | `Resolve-DnsName <NAME>` | DNS resolution |
| `Test-NetConnection <HOST> -Port <PORT>` | Test TCP port | `Get-NetTCPConnection -State Listen` | Listening TCP ports |

**Network diagnostic order:** adapter/link → IP/subnet/gateway/DNS → gateway reachability → external IP → DNS lookup → required TCP port. A failed ping does not prove a host is offline; ICMP may be blocked. Do not run `ipconfig /release` over remote access unless alternative access exists.

Repair Windows system files in an elevated terminal:

```cmd
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow
```

Check updates with `Get-HotFix` or open `ms-settings:windowsupdate`. Use maintenance windows, plan restarts and verify services afterward.

## 2.7 Windows firewall and hardening

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `Get-NetFirewallProfile` | Firewall profile state | `Get-NetFirewallRule -Enabled True` | Enabled rules |
| `wf.msc` | Advanced firewall rules | `Get-MpComputerStatus` | Defender status |

Keep firewall and approved endpoint protection enabled. Block unsolicited inbound traffic by default where appropriate; allow only required services. Restrict RDP to trusted networks and approved users. Patch Windows/apps, use MFA, review local admins and logs, disable unnecessary services and maintain tested backups.

## 2.8 Windows quick troubleshooting

| Symptom | First checks | Symptom | First checks |
|---|---|---|---|
| Slow/high CPU | Task Manager, `resmon`, `Get-Process` | Service unavailable | `Get-Service`, Event Viewer |
| Application crash | Application log, Reliability Monitor | No network | `ipconfig /all`, adapter, gateway |
| IP works, name fails | `Resolve-DnsName`, DNS cache | Port unavailable | `Test-NetConnection` |
| Disk full | `Get-Volume`, Disk Management | System file corruption | DISM, then SFC |
| Suspicious login | Security log, IDs 4624/4625 | Unexpected process | Process path, signature, parent |

---

# 3. Windows Server and Active Directory

## 3.1 Common server roles

| Role | Purpose | Role | Purpose |
|---|---|---|---|
| AD DS | Directory and identity | DNS | Name resolution |
| DHCP | Automatic IP configuration | File Services | File shares/storage |
| Hyper-V | Virtual machines | IIS | Web hosting |
| Remote Desktop Services | Remote desktop/apps |  |  |

`Get-WindowsFeature` lists installed/available roles. Example installation: `Install-WindowsFeature DNS -IncludeManagementTools`. Installing AD DS is not the same as promoting a server to a domain controller.

## 3.2 AD concepts

| Term | Meaning | Term | Meaning |
|---|---|---|---|
| Forest | Top-level AD structure | Domain | Directory/administrative boundary |
| OU | Organises objects; GPO/delegation scope | DC | Server hosting AD DS/authentication |
| Global Catalog | Locates objects across forest | Site | Represents network topology/replication |
| Trust | Allows authentication across domains/forests | GPO | Central user/computer configuration |

Typical structure: forest `company.local` → domain → OUs for Users, Computers, Servers and Groups.

## 3.3 Domain controller deployment and health

Basic sequence: install Server → configure static IP and AD DNS → name server → install AD DS role → promote as new/existing DC → verify DNS, authentication and replication. Production environments normally use multiple DCs for availability.

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "company.local"   # New forest; planned deployment only
```

Promotion requires planning, credentials and restart. Do not run the forest command on an existing production server without an approved design.

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `dcdiag` | DC health tests | `dcdiag /v` | Detailed DC diagnostics |
| `repadmin /replsummary` | Replication summary | `repadmin /showrepl` | Replication partners/status |
| `echo %LOGONSERVER%` | DC used by current logon | `Get-ADDomainController -Filter *` | List DCs |

## 3.4 Users and groups

Tools: `dsa.msc` (AD Users and Computers), `gpmc.msc` (Group Policy), PowerShell ActiveDirectory module.

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `Get-ADUser -Filter *` | List users | `Get-ADUser <USER> -Properties *` | Inspect user properties |
| `Disable-ADAccount <USER>` | Disable account | `Enable-ADAccount <USER>` | Enable account |
| `Unlock-ADAccount <USER>` | Unlock account | `Get-ADGroupMember <GROUP>` | List group members |
| `Add-ADGroupMember <GROUP> -Members <USER>` | Add user to group | `Get-ADGroupMember "Domain Admins"` | Review privileged membership |

Set an account password safely:

```powershell
$pw = Read-Host "New password" -AsSecureString
Set-ADAccountPassword -Identity <USER> -Reset -NewPassword $pw
```

Create a user/group only after confirming the correct OU, naming convention and account requirements. Use groups to assign access rather than granting permissions user by user.

Group scopes: **Global** (organise users from a domain), **Domain Local** (assign permissions to resources in a domain), **Universal** (members across domains in a forest).

## 3.5 Group Policy

GPO centrally configures password/account policy, firewall, updates, security settings, scripts and mapped drives. Typical processing order is **LSDOU**: Local → Site → Domain → OU (parent to child). Inheritance, link order and security filtering can change the result.

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `gpupdate /force` | Refresh policies | `gpresult /r` | Show applied policies |
| `gpresult /h C:\Temp\gpo.html` | Create HTML report | `gpmc.msc` | Manage GPOs |

If a GPO does not apply, check OU placement, filtering, inheritance, replication and whether the setting is under User or Computer Configuration.

## 3.6 DNS and DHCP in a domain

DNS is essential for AD; clients use DNS SRV records to locate domain controllers.

| Record | Purpose | Record | Purpose |
|---|---|---|---|
| A / AAAA | Name → IPv4 / IPv6 | CNAME | Alias |
| MX | Mail server | PTR | Reverse lookup |
| SRV | Locate services/DCs |  |  |

```powershell
Resolve-DnsName company.local
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.company.local
```

DHCP provides address, subnet, gateway, DNS and lease details. DORA = **Discover → Offer → Request → Acknowledge**. Check service/authorisation, active scope, free leases, correct options and DHCP relay/IP helper between VLANs. DHCP need not run on a DC.

## 3.7 Authentication, ports and domain join

- **Kerberos:** AD authentication using tickets; relies on DNS and accurate time.
- **LDAP:** Protocol for querying/managing directory information.

| Port | Service | Port | Service |
|---:|---|---:|---|
| TCP/UDP 88 | Kerberos | TCP/UDP 389 | LDAP |
| TCP 636 | LDAPS | TCP/UDP 53 | DNS |
| UDP 123 | NTP | TCP 445 | SMB |

Before domain join, configure AD DNS, verify DC reachability and time, and use authorised credentials:

```powershell
Add-Computer -DomainName "company.local" -Credential "COMPANY\AdminUser" -Restart
```

## 3.8 AD troubleshooting and security

| Symptom | First checks | Symptom | First checks |
|---|---|---|---|
| Cannot join domain | DNS, DC reachability, time, credentials | Login fails | Enabled/locked account, password, DC, time |
| Repeated lockout | Security logs, saved credentials, tasks/services | GPO missing | `gpresult`, OU, filtering, replication |
| DC not found | DNS SRV records, DNS server, network | Replication issue | `repadmin`, DNS, connectivity |
| Kerberos fails | Time, DNS, account/SPN | Slow logon | DNS, DC, GPO, network |

Useful checks: `ipconfig /all`, `nslookup company.local`, `dcdiag`, `repadmin /replsummary`, `gpresult /r`, `Get-ADDomain`, `Get-ADForest`.

**AD security:** separate privileged accounts; minimise Domain Admin membership; use MFA where supported; patch and restrict DCs; monitor logons and privileged group changes; review stale accounts/computers; maintain and test AD-aware backups. DC compromise can compromise domain identity and access.

---

# 4. Networking Fundamentals and Protocols

## 4.1 Models and encapsulation

| OSI layer | Name | Examples / purpose |
|---:|---|---|
| 7 | Application | HTTP, DNS, SMTP, SSH |
| 6 | Presentation | Data format, encryption, compression |
| 5 | Session | Session management |
| 4 | Transport | TCP, UDP, ports |
| 3 | Network | IPv4/IPv6, ICMP, routing |
| 2 | Data Link | Ethernet, Wi-Fi, VLAN, MAC |
| 1 | Physical | Copper, fiber, radio |

TCP/IP combines OSI 5–7 as Application, 4 as Transport, 3 as Internet and 1–2 as Network Access.

Encapsulation: **data → TCP segment/UDP datagram → IP packet → Ethernet frame → bits**. IP addresses identify endpoints across networks; MAC addresses deliver frames on the local segment. Routers replace Layer 2 framing at each hop.

## 4.2 TCP, UDP, Ethernet and ARP

| TCP | UDP |
|---|---|
| Connection-oriented; reliable and ordered | Connectionless; no built-in delivery guarantee |
| Acknowledgements and retransmissions | Lower overhead |
| Common for web, SSH and file transfer | Common for DNS, voice/video and latency-sensitive traffic |

TCP handshake: **SYN → SYN-ACK → ACK**. `FIN` closes normally; `RST` resets a connection.

Ethernet is mainly Layer 2. A MAC address is normally 48-bit hexadecimal. **ARP** maps IPv4 addresses to MAC addresses on a local network; IPv6 uses **NDP**, not ARP.

## 4.3 IPv4 subnetting

IPv4 uses 32-bit addresses. Prefix length identifies the network portion.

| Prefix | Subnet mask | Prefix | Subnet mask |
|---:|---|---:|---|
| `/24` | `255.255.255.0` | `/25` | `255.255.255.128` |
| `/26` | `255.255.255.192` | `/27` | `255.255.255.224` |
| `/28` | `255.255.255.240` | `/30` | `255.255.255.252` |

Typical usable host count: `2^(32-prefix) - 2` (special cases include `/31` and `/32`).

Example: `192.168.10.77/26` → network `192.168.10.64`, broadcast `192.168.10.127`, usable `192.168.10.65–126`.

| Range | Purpose | Range | Purpose |
|---|---|---|---|
| `10.0.0.0/8` | Private | `172.16.0.0/12` | Private |
| `192.168.0.0/16` | Private | `127.0.0.0/8` | Loopback |
| `169.254.0.0/16` | Link-local/APIPA | `0.0.0.0/0` | Default route |

## 4.4 IPv6, DNS, DHCP and NAT

IPv6 uses 128-bit hexadecimal addresses. Key ranges: `::1` loopback, `fe80::/10` link-local, `fc00::/7` unique local, `2000::/3` global unicast, `::/0` default route. NDP replaces ARP. Address configuration can use SLAAC or DHCPv6. Review IPv6 firewall rules as well as IPv4.

- **DNS:** resolves names; common records A, AAAA, CNAME, MX, PTR, SRV.
- **DHCP:** assigns address, prefix/mask, gateway, DNS and lease; DORA = Discover, Offer, Request, Acknowledge.
- **NAT:** translates IP addresses; PAT/NAT overload lets multiple internal devices share a public IP using ports. NAT is not a firewall.
- **Default gateway:** router/interface used for destinations outside the local subnet.
- **Routing:** selects the most specific matching route. Default routes: `0.0.0.0/0` and `::/0`.

## 4.5 Important ports

| Service | Port/protocol | Service | Port/protocol |
|---|---|---|---|
| FTP control | 21/TCP | SSH/SFTP | 22/TCP |
| Telnet | 23/TCP | SMTP | 25/TCP |
| DNS | 53/TCP,UDP | DHCP | 67/68 UDP |
| TFTP | 69/UDP | HTTP | 80/TCP |
| Kerberos | 88/TCP,UDP | POP3 | 110/TCP |
| NTP | 123/UDP | IMAP | 143/TCP |
| SNMP | 161/162 UDP | LDAP | 389/TCP,UDP |
| HTTPS | 443/TCP | SMB | 445/TCP |
| SMTPS/submission | 465/587 TCP | LDAPS | 636/TCP |
| IMAPS | 993/TCP | POP3S | 995/TCP |
| RDP | 3389/TCP,UDP |  |  |

Prefer encrypted protocols such as SSH, HTTPS and LDAPS over Telnet, FTP and unencrypted management services.

---

# 5. Switching, Routing and Network Infrastructure

## 5.1 Devices and VLANs

| Device | Function | Device | Function |
|---|---|---|---|
| Switch | Forwards LAN frames using MAC addresses | Router | Connects IP networks |
| Layer 3 switch | Switching and inter-VLAN routing | Firewall | Filters traffic |
| Access point | Wireless LAN access | Modem/ONT | Provider access connection |

A **VLAN** creates a separate Layer 2 broadcast domain. Commonly separate users, servers, guests and management.

- **Access port:** one VLAN, usually for an endpoint.
- **Trunk port:** carries multiple VLANs, usually using 802.1Q tags, between network devices.

Cisco IOS examples (syntax varies by platform):

```text
show vlan brief
show interfaces status
show interfaces trunk
show mac address-table
```

```text
vlan 10
 name USERS
interface gi0/1
 switchport mode access
 switchport access vlan 10
```

Trunk example:

```text
interface gi0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

## 5.2 Switching, STP and inter-VLAN routing

A switch learns source MAC addresses and maps them to ports. Compare:
- **MAC table:** MAC address → switch port.
- **ARP table:** IPv4 address → MAC address.

`show mac address-table` displays learned MACs.

**STP (Spanning Tree Protocol)** prevents Layer 2 loops by blocking redundant paths while preserving backup links. Check root bridge and port states with `show spanning-tree`. Avoid changing STP without understanding the topology.

Devices in different VLANs need Layer 3 routing:
- **Router-on-a-stick:** router subinterfaces over a trunk.
- **Layer 3 switch:** SVIs provide VLAN gateways.

Example SVI:

```text
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

Verify VLAN membership, trunk allowance, gateway, routing and ACL/firewall rules.

## 5.3 Routing and link aggregation

- **Static route:** manually configured; suitable for small/predictable networks.
- **OSPF:** link-state dynamic routing within an organisation/autonomous system.
- **BGP:** path-vector routing between autonomous systems; used on the internet.
- **RIP:** older distance-vector protocol with limited scalability.

Useful checks: `show ip route`, `show ip protocols`, `show ip interface brief`.

**EtherChannel** combines physical links into one logical link for capacity and redundancy. LACP is commonly used for negotiation. Verify members have compatible speed, duplex, VLAN and trunk settings.

```text
show etherchannel summary
show interfaces port-channel
```

## 5.4 Physical infrastructure

| Item | Practical checks |
|---|---|
| Copper Ethernet | Cable category, distance, connector, link LEDs |
| Fiber | Single-mode vs multimode, connector cleanliness, optics and distance |
| SFP/SFP+ | Speed, wavelength, fiber type and compatibility at both ends |
| PoE | Switch power budget and endpoint power requirement |

For a down link, check power, cable/transceiver, port state, speed/duplex and interface errors.

## 5.5 Cisco IOS verification and safe changes

| Command | Purpose | Command | Purpose |
|---|---|---|---|
| `show running-config` | Active configuration | `show startup-config` | Saved configuration |
| `show interfaces status` | Port state | `show interfaces` | Errors and interface details |
| `show ip interface brief` | Interface/IP summary | `show vlan brief` | VLAN membership |
| `show interfaces trunk` | Trunk status | `show spanning-tree` | STP state |
| `show mac address-table` | Learned MACs | `show ip route` | Routing table |
| `show etherchannel summary` | Port-channel state | `show logging` | Device logs |

Before a network change: confirm approval and impact; back up configuration; prepare rollback; change one thing; verify interfaces/VLANs/routes/connectivity; inspect logs; save only after successful verification.

---

# 6. Network and System Troubleshooting

## 6.1 Repeatable method

1. Identify symptom, time, affected user/device/service.
2. Determine scope: one host, VLAN, site or whole environment.
3. Check recent changes.
4. Collect command output, logs and monitoring evidence.
5. Isolate one layer/component at a time.
6. Make the smallest authorised, reversible change.
7. Verify the original function and inspect for new errors.
8. Document cause, action, result and rollback.

Do not start by rebooting, disabling firewalls or flushing configuration.

## 6.2 Network checks in order

| Step | What to check | Useful commands |
|---|---|---|
| 1. Physical | Power, cable, Wi-Fi, link LEDs, adapter | `ip link`; `Get-NetAdapter` |
| 2. IP config | Address, prefix/mask, gateway, DNS | `ip addr`; `ipconfig /all` |
| 3. Gateway | Local subnet and gateway reachability | `ping <GATEWAY>` |
| 4. Routing | Default route, next hop, return path | `ip route`; `route print`; `tracert` |
| 5. DNS | Resolver and hostname lookup | `getent hosts`; `Resolve-DnsName`; `nslookup` |
| 6. Service | Listener, host firewall, network ACL | `ss -lntup`; `Test-NetConnection host -Port <P>` |
| 7. Path | Where traffic stops or loss appears | `tracepath`; `tracert`; `pathping` |
| 8. Packets/logs | Capture only if necessary; inspect relevant logs | `tcpdump`; Wireshark; system/device logs |

`169.254.x.x` commonly indicates IPv4 DHCP failure. A failed ping does not prove a host is offline—ICMP may be filtered. Packet captures can contain sensitive data; capture only authorised traffic.

## 6.3 Common symptoms

| Symptom | Likely checks | Symptom | Likely checks |
|---|---|---|---|
| No network | Link, adapter, VLAN, DHCP | `169.254.x.x` | DHCP server/scope/relay |
| Gateway unreachable | Link, VLAN, subnet, gateway | IP works, name fails | DNS server/records/resolver |
| One port unavailable | Service, host firewall, ACL | Intermittent link | Cable, duplex, congestion, Wi-Fi |
| One host affected | Local config/adapter/firewall | Many hosts affected | Shared switch/router/DNS/DHCP |
| VLANs cannot communicate | Inter-VLAN routing, ACL | Slow application | Loss, latency, server resource, DNS |

## 6.4 System checks

| Problem | Linux | Windows |
|---|---|---|
| High CPU | `top`; `ps aux --sort=-%cpu \| head` | Task Manager; `Get-Process` sorted by CPU |
| High memory | `free -h`; `top` | Task Manager; sort by WorkingSet |
| Disk full | `df -h`; `du -xhd1 /` | `Get-Volume`; Disk Management |
| Service failure | `systemctl status`; `journalctl -u` | `Get-Service`; Event Viewer |
| Deleted log still using space | `lsof +L1` | Identify open file/process; use approved tools |

Do not delete application data or logs blindly. Check ownership, retention requirements and what consumes the space before acting.

## 6.5 Troubleshooting checklist

- [ ] Record host, user, time and exact error.
- [ ] Check whether the issue affects other devices.
- [ ] Record IP, route, adapter and service state.
- [ ] Review recent changes and relevant logs.
- [ ] Confirm authorisation and backup config before editing.
- [ ] Change one thing at a time; verify and document.

---

# 7. Cybersecurity Fundamentals and System Hardening

## 7.1 Core principles and threats

| Principle | Practical meaning |
|---|---|
| Confidentiality | Only authorised people access information |
| Integrity | Prevent unauthorised changes |
| Availability | Keep systems/data accessible |
| Least privilege | Grant only necessary access |
| Defense in depth | Multiple independent controls |
| Zero Trust | Verify access explicitly; do not trust network location alone |

Common threats: malware/ransomware; phishing; password spraying/brute force; stolen credentials; privilege escalation; lateral movement; misconfiguration; denial of service.

## 7.2 Identity and access

- Use individual accounts; avoid shared administrator accounts.
- Use MFA, especially for privileged and remote access.
- Use standard accounts for daily work.
- Restrict and review privileged group membership.
- Disable/remove accounts no longer required.
- Use unique passwords and an approved password manager.
- Review service accounts and their permissions.

## 7.3 Host hardening

| Linux | Windows |
|---|---|
| Patch OS/packages: `apt update && apt upgrade` | Patch OS and applications through approved process |
| Review sudo: `sudo -l` | Review local Administrators membership |
| Validate SSH: `sshd -t` | Check Defender: `Get-MpComputerStatus` |
| Review firewall: `ufw status`; `nft list ruleset` | Review firewall: `Get-NetFirewallProfile` |
| Restrict SSH to authorised users/networks; prefer keys | Restrict RDP; require VPN/gateway and MFA where supported |
| Avoid `777`; remove unused services/accounts | Disable unnecessary services and remote access |

Keep endpoint protection and host firewalls enabled. Avoid direct root SSH login where appropriate. Monitor authentication and privileged activity.

## 7.4 Vulnerability and patch management

1. Maintain system/software inventory.
2. Identify missing updates and known vulnerabilities.
3. Prioritise exposed, critical and actively exploited systems.
4. Test where appropriate.
5. Deploy during approved maintenance windows.
6. Plan restarts and rollback.
7. Verify patch level and service health.
8. Track exceptions and unresolved risks.

## 7.5 Logs, hashes and backups

- Monitor authentication, privileged actions, service/application errors, firewall and endpoint alerts.
- Protect logs from unauthorised modification; retain according to policy.
- Hashes help compare file contents; a changed hash is not proof of malware.

| Command | Explanation | Command | Explanation |
|---|---|---|---|
| `sha256sum <FILE>` | Linux SHA-256 hash | `Get-FileHash <FILE> -Algorithm SHA256` | Windows SHA-256 hash |

**3-2-1 backup rule:** 3 copies, 2 different storage types, 1 off-site or isolated copy. Protect backups from ransomware and test restoration regularly.

## 7.6 Hardening checklist

- [ ] Patch operating systems and applications.
- [ ] Remove unnecessary accounts, services and software.
- [ ] Apply least privilege and MFA.
- [ ] Restrict remote administration.
- [ ] Enable host firewalls and endpoint protection.
- [ ] Review privileged groups and permissions.
- [ ] Protect and monitor logs.
- [ ] Maintain isolated backups and test restoration.
- [ ] Document exceptions and configuration changes.

---

# 8. Network Security

## 8.1 Firewalls and filtering

Firewalls allow/block traffic based on source, destination, protocol, port and connection state.

- **Stateless:** evaluates individual packets.
- **Stateful:** tracks connection state.
- **NGFW:** may add application awareness, intrusion prevention and other controls.
- **Host firewall:** protects an individual endpoint/server.

Default-deny unsolicited inbound traffic where appropriate. Allow only required services, restrict management access, log relevant events and review/remove obsolete rules.

## 8.2 Segmentation and remote access

Separate user, server, guest, management and critical networks. VLANs provide Layer 2 separation; ACLs/firewalls control traffic between segments. A VLAN alone does not block routed traffic.

| Access method | Minimum practical controls |
|---|---|
| SSH | Keys where appropriate, restrict users/source networks, patch and monitor |
| RDP | Do not expose directly to internet; use VPN/approved gateway, MFA and access restrictions |
| VPN | Approved encrypted client/protocol, MFA and access only to required networks/services |

## 8.3 Common network controls

| Control | Purpose | Control | Purpose |
|---|---|---|---|
| ACL | Permit/deny traffic | IDS | Detect and alert |
| IPS | Detect and block selected traffic | DHCP snooping | Help block rogue DHCP servers |
| Dynamic ARP Inspection | Help prevent ARP spoofing | Port security | Restrict MACs on switch ports |
| 802.1X | Authenticate before network access | SNMPv3 | Secure device monitoring |
| DNS filtering | Block/alert on domains | Egress filtering | Restrict outbound traffic |

## 8.4 Network security checklist

- [ ] Separate user, server, guest and management networks.
- [ ] Restrict inter-VLAN traffic.
- [ ] Disable unused switch ports.
- [ ] Use secure management protocols.
- [ ] Restrict SSH, RDP and device administration.
- [ ] Use MFA for remote access.
- [ ] Prefer SNMPv3.
- [ ] Monitor firewall and network-device logs.
- [ ] Review rules and update device firmware.
- [ ] Back up network-device configurations.

---

# 9. SOC, Security Monitoring and Incident Response

## 9.1 SOC terms

| Term | Meaning |
|---|---|
| SOC | Team/function monitoring and investigating security activity |
| SIEM | Centralises and correlates logs/events |
| EDR | Endpoint detection, investigation and response |
| IOC | Indicator of compromise, e.g. hash, IP or domain |
| IOA | Indicator based on suspicious behaviour |
| TTP | Adversary tactics, techniques and procedures |
| False positive | Alert that is not a real incident |
| False negative | Malicious activity not detected |

An IOC is an investigative lead, not proof by itself.

## 9.2 Alert triage

| Check | Questions |
|---|---|
| Alert | What rule fired, when, severity, source and reason? |
| Asset | Which host/IP/user? Who owns it? How critical is it? |
| Process | Path, command line, hash, signature, parent/child process? |
| Account | Who ran it? Privileges? Logon type? Expected activity? |
| Persistence | New service, scheduled task, startup or registry change? |
| Network | Remote IP/domain, port, DNS query or unusual connection? |
| Scope | Other affected hosts/accounts or related alerts? |
| Decision | Benign, suspicious, confirmed or escalate? |

Do not judge a process by its name alone. Check path, signature, hash, command line, parent process and behaviour.

## 9.3 Important Windows events

| Event ID | Meaning | Event ID | Meaning |
|---:|---|---:|---|
| 4624 | Successful logon | 4625 | Failed logon |
| 4648 | Explicit credentials used | 4672 | Special privileges assigned |
| 4688 | Process creation (if audited) | 4697 / 7045 | Service installed |
| 4698 | Scheduled task created | 4720 | User created |
| 4728 | Added to global security group | 4732 | Added to local security group |

Example query:

```powershell
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4624,4625,4688,4697,4698,4720,4728,4732
} -MaxEvents 100
```

Check account, timestamp, logon type, source address and surrounding activity. Event availability depends on audit settings and retention.

## 9.4 Sysmon and investigation tools

Sysmon provides extra Windows telemetry; event coverage depends on configuration.

| Sysmon ID | Activity | Sysmon ID | Activity |
|---:|---|---:|---|
| 1 | Process creation | 3 | Network connection |
| 7 | Image/module loaded | 10 | Process access |
| 11 | File created | 12–14 | Registry events |
| 22 | DNS query |  |  |

| Tool | Use | Tool | Use |
|---|---|---|---|
| Process Explorer | Process tree/details | Autoruns | Startup/persistence |
| TCPView | Process network connections | Sysmon | Endpoint telemetry |
| Defender/EDR | Detection and response | Wireshark/tcpdump | Packet analysis |
| SIEM | Central event search/correlation |  |  |

## 9.5 MITRE ATT&CK examples

| Technique | ID | Example |
|---|---|---|
| Command and Scripting Interpreter | T1059 | Shells/scripts |
| Scheduled Task | T1053.005 | Task-based persistence/execution |
| Windows Service | T1543.003 | Create/modify service |
| Registry Run Keys | T1547.001 | Startup persistence |
| OS Credential Dumping | T1003 | Attempt to obtain credentials |
| Remote Services | T1021 | Remote access/lateral movement |
| Impair Defenses | T1562.001 | Disable/weaken security tools |

A technique mapping alone does not prove malicious activity; assess context and evidence.

## 9.6 Incident response

Follow the organisation's incident response plan:

1. **Identify:** confirm alert and affected systems/accounts.
2. **Preserve:** record timestamps, logs, hashes, command lines, processes and connections.
3. **Assess:** determine impact and business criticality.
4. **Contain:** approved isolation, IOC blocking or session revocation.
5. **Escalate:** notify incident responders and stakeholders.
6. **Eradicate/recover:** remove cause, restore safely and verify integrity.
7. **Monitor:** look for persistence, lateral movement and recurrence.
8. **Document:** actions, evidence, decisions and lessons learned.

**Do not:** clear logs; disable EDR to dismiss an alert; run suspected malware; delete evidence before collection unless immediate containment requires it; reset passwords without considering sessions/tokens; isolate critical production systems without authorisation and impact assessment.

**Sequence:** Observe → preserve context → verify → assess impact → escalate or take approved action.

---

# 10. Essential Administration Procedures and Reference

## 10.1 Before and after a change

| Before | After |
|---|---|
| Confirm request and authorisation | Verify configuration and service |
| Identify affected systems/users | Test dependent services |
| Check dependencies and impact | Check logs and monitoring |
| Record current configuration | Confirm security controls remain enabled |
| Back up relevant data/config | Save configuration if appropriate |
| Prepare rollback and maintenance window | Document result and outstanding risks |
| Notify affected teams | Close out change record |

## 10.2 Cross-platform command reference

| Task | Linux | Windows |
|---|---|---|
| Current user | `whoami` | `whoami` |
| Hostname | `hostname` | `hostname` |
| IP configuration | `ip addr` | `ipconfig /all` |
| Routing | `ip route` | `route print` |
| DNS lookup | `getent hosts <NAME>` | `nslookup <NAME>` |
| Connectivity | `ping <HOST>` | `ping <HOST>` |
| Trace route | `tracepath <HOST>` | `tracert <HOST>` |
| Listening ports | `ss -lntup` | `netstat -ano` |
| Processes | `ps aux` | `Get-Process` |
| Services | `systemctl` | `Get-Service` |
| Logs | `journalctl` | Event Viewer / `Get-WinEvent` |
| Disk space | `df -h` | `Get-Volume` |
| File hash | `sha256sum <FILE>` | `Get-FileHash <FILE> -Algorithm SHA256` |

## 10.3 Incident/change record

Record: date/time and timezone; hostname/IP/environment; ticket/incident ID; description; evidence (logs, commands, hashes); action and operator; result; rollback method; follow-up risks.

## 10.4 Operational rules

1. Diagnose before changing configuration.
2. Use least privilege.
3. Do not expose management interfaces unnecessarily.
4. Keep systems patched and monitored.
5. Back up configurations and important data.
6. Test restoration, not just backup creation.
7. Preserve evidence during incidents.
8. Document changes and verify results.
9. Follow organisational procedures and escalation paths.
10. Understand commands before running them with elevated privileges.

**Core principle:** Make controlled, authorised and reversible changes. Verify the result and leave a clear record for the next administrator.
