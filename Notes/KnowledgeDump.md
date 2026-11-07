# 1. Linux Administration

**Scope:** Debian-based Linux. Commands may differ on other distributions.

## 1.1 Essential Files and Directories

| Path | Purpose |
|---|---|
| `/` | Filesystem root |
| `/etc` | System configuration |
| `/home` | User home directories |
| `/root` | Root user's home |
| `/var/log` | System and application logs |
| `/var` | Variable data, logs and caches |
| `/tmp` | Temporary files |
| `/usr` | Programs and libraries |
| `/opt` | Optional/third-party software |
| `/boot` | Kernel and boot files |
| `/dev` | Device files |
| `/proc`, `/sys` | Process, kernel and hardware information |

Important files: `/etc/passwd`, `/etc/group`, `/etc/fstab`, `/etc/ssh/sshd_config`.

## 1.2 Essential Terminal Commands

```bash
pwd                         # Current directory
ls -lah                      # Files, including hidden files
cd /etc; cd ..; cd ~         # Change directory
find /var/log -name "*.log"  # Find files
file <FILE>                  # Identify file type

cp source destination        # Copy
cp -a source_dir backup_dir  # Copy and preserve attributes
mv old new                   # Move/rename
mkdir -p /opt/app/logs       # Create directory tree
rm -i <FILE>                 # Delete with confirmation
tar -czf backup.tar.gz dir/  # Create archive
tar -xzf backup.tar.gz       # Extract archive
````

Caution: `rm -rf` deletes recursively without confirmation. Verify paths carefully, especially when using `sudo`.

### Read, search and process text

```
less <FILE>                  # Read a file
head -n 20 <FILE>             # First 20 lines
tail -n 50 <FILE>             # Last 50 lines
tail -f /var/log/syslog       # Follow a log
grep -n "error" <FILE>        # Search text
grep -Rni "failed" /var/log/  # Recursive case-insensitive search
sort file.txt | uniq -c       # Count repeated lines
cut -d: -f1 /etc/passwd       # Extract first field
```

### Pipes and redirection

| Syntax          | Meaning                        |
| --------------- | ------------------------------ |
| `cmd1 \\| cmd2` | Send output to another command |
| `>`             | Write/overwrite a file         |
| `>>`            | Append to a file               |
| `2>`            | Redirect errors                |

```
ip addr > network.txt
command >> output.log
command 2> errors.log
```

Be careful: `>` overwrites existing file contents.

## 1.3 Users and Permissions

```
whoami
id
groups
sudo -l

sudo adduser anna
sudo passwd anna
sudo usermod -aG sudo anna
sudo passwd -l anna
getent passwd anna
```

Use `usermod -aG` when adding a supplementary group. Omitting `-a` can replace existing supplementary group memberships.

### Permission model

- `u` = owner, `g` = group, `o` = others.
- `r` = read (4), `w` = write (2), `x` = execute (1).

| Mode  | Meaning                         |
| ----- | ------------------------------- |
| `755` | Owner full; others read/execute |
| `750` | Owner full; group read/execute  |
| `700` | Owner only                      |
| `644` | Owner read/write; others read   |
| `640` | Owner read/write; group read    |
| `600` | Owner read/write only           |

```
chmod 640 file.txt
chmod u+x script.sh
chown anna:developers file.txt
chgrp developers file.txt
getfacl file.txt
setfacl -m u:anna:r file.txt
umask
```

Avoid `chmod 777`; it gives everyone read, write and execute access. Use ACLs when standard owner/group/other permissions are insufficient.

## 1.4 Packages, Processes and Services

### APT package management

```
sudo apt update                 # Refresh package indexes
sudo apt upgrade                # Upgrade installed packages
sudo apt install <PACKAGE>
sudo apt remove <PACKAGE>
sudo apt purge <PACKAGE>
sudo apt autoremove
apt search <NAME>
apt show <PACKAGE>
dpkg -l                         # Installed packages
dpkg -S /path/to/file            # Find owning package
```

`apt update` refreshes package indexes; it does not install updates.

### Processes and services

```
ps aux
pgrep -af <NAME>
top
free -h
uptime
kill -TERM <PID>                # Request graceful termination
kill -KILL <PID>                # Force termination; last resort

systemctl status <SERVICE>
sudo systemctl start <SERVICE>
sudo systemctl stop <SERVICE>
sudo systemctl restart <SERVICE>
sudo systemctl enable <SERVICE>
sudo systemctl enable --now <SERVICE>
systemctl --failed
```

Use `TERM` before `KILL`. Do not stop or disable an unfamiliar service without checking its purpose and dependencies.

## 1.5 Logs and Storage

### Logs and boot diagnostics

```
journalctl -b                   # Current boot
journalctl -p err -b            # Errors from current boot
journalctl -u <SERVICE>         # Service logs
journalctl -u <SERVICE> -f      # Follow service logs
journalctl --since "1 hour ago"
dmesg -T                        # Kernel messages
systemctl --failed
```

Common log locations: `/var/log/syslog`, `/var/log/auth.log`, `/var/log/kern.log` (availability depends on logging configuration).

### Disk and filesystem checks

```
lsblk -f
df -h                           # Filesystem free space
du -sh <PATH>                   # Directory size
findmnt
sudo fdisk -l
sudo blkid
```

Mount a filesystem:

```
sudo mkdir -p /mnt/data
sudo mount /dev/sdb1 /mnt/data
findmnt /mnt/data
sudo umount /mnt/data
```

For automatic mounting, use a filesystem UUID in `/etc/fstab`. Validate changes before rebooting:

```
sudo findmnt --verify
sudo mount -a
```

An incorrect `/etc/fstab` entry can cause boot or mount problems.

LVM: PV (physical volume) → VG (volume group) → LV (logical volume).

```
sudo pvs
sudo vgs
sudo lvs
```

When a disk is full:

```
df -h
sudo du -xhd1 /
sudo lsof +L1
```

Do not delete logs or application data blindly. Identify what is consuming space first.

## 1.6 Linux Networking

```
ip -br addr                    # IP addresses/interfaces
ip link                        # Interface state
ip route                       # Routing table
ip route get 1.1.1.1            # Selected route
ip neigh                       # ARP/NDP neighbours

ping -c 4 1.1.1.1
tracepath example.com
getent hosts example.com       # Name resolution
resolvectl status              # DNS configuration
ss -lntup                       # Listening TCP/UDP sockets
ss -s                           # Socket statistics
curl -I https://example.com    # HTTP response headers
sudo tcpdump -ni eth0 host 10.0.0.5
```

Packet captures may contain sensitive information. Capture only traffic you are authorized to inspect.

## 1.7 SSH, Firewall and Hardening

```
ssh user@server
scp file.txt user@server:/tmp/
sudo sshd -t                   # Validate SSH configuration
sudo systemctl reload ssh
sudo ufw status verbose
sudo ufw allow from 10.10.0.0/24 to any port 22 proto tcp
sudo ufw deny 23/tcp
sudo nft list ruleset
```

Before changing SSH or firewall settings remotely:

1. Confirm the required management access is allowed.
2. Validate configuration.
3. Keep the current remote session open.
4. Test a second connection before closing the first.

Linux hardening checklist

- Patch the OS and applications.
- Remove unnecessary accounts and services.
- Use individual accounts and least privilege.
- Prefer SSH keys; restrict SSH to trusted networks.
- Disable direct root SSH login where appropriate.
- Avoid world-writable permissions.
- Enable and review firewall rules.
- Monitor authentication and system logs.
- Maintain backups and test restoration.

## 1.8 Linux Quick Troubleshooting

| Symptom              | First checks                                       |
| -------------------- | -------------------------------------------------- |
| Slow server          | `uptime`, `top`, `free -h`                         |
| Disk full            | `df -h`, `du -xhd1 /`, `lsof +L1`                  |
| Service unavailable  | `systemctl status`, `journalctl -u`                |
| Interface down       | `ip -br addr`, `ip link`                           |
| No connectivity      | `ip route`, ping gateway                           |
| IP works, name fails | `resolvectl status`, `getent hosts`                |
| Port unavailable     | `ss -lntup`, firewall, remote test                 |
| Boot problem         | `journalctl -b`, `dmesg -T`                        |
| SSH fails            | Service status, auth logs, firewall and SSH config |

# 2. Windows Administration

Scope: Windows 10/11 and Windows Server. Distinguish CMD from PowerShell. Run elevated commands only when required; domain policy or Intune may override local settings.

## 2.1 Essential Windows Tools

Open these using `Win + R`:

| Command        | Purpose                                   |
| -------------- | ----------------------------------------- |
| `taskmgr`      | Processes and performance                 |
| `services.msc` | Services                                  |
| `eventvwr.msc` | Event Viewer                              |
| `devmgmt.msc`  | Device Manager                            |
| `diskmgmt.msc` | Disk Management                           |
| `ncpa.cpl`     | Network adapters                          |
| `wf.msc`       | Advanced Windows Firewall                 |
| `sysdm.cpl`    | System/domain settings                    |
| `resmon`       | Resource Monitor                          |
| `perfmon /rel` | Reliability Monitor                       |
| `taskschd.msc` | Scheduled Tasks                           |
| `compmgmt.msc` | Computer Management                       |
| `mstsc`        | Remote Desktop                            |
| `gpedit.msc`   | Local Group Policy (edition-dependent)    |
| `secpol.msc`   | Local Security Policy (edition-dependent) |

## 2.2 CMD and PowerShell Essentials

### CMD

```
whoami /all
hostname
systeminfo
tasklist
tasklist /svc
driverquery /v
net user
net localgroup
net localgroup Administrators
taskkill /PID <PID> /T
```

Use `/F` with `taskkill` only when force termination is necessary. Verify the PID first.

### PowerShell

```
Get-ComputerInfo
Get-HotFix
Get-Service
Get-Process
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember -Group 'Administrators'
Get-Disk
Get-Partition
Get-Volume
```

Filter and sort objects:

```
Get-Process |
  Sort-Object CPU -Descending |
  Select-Object -First 10

Get-Service |
  Where-Object Status -ne 'Running'
```

Useful pipeline commands: `Where-Object` (filter), `Select-Object` (choose fields), `Sort-Object` (sort), `Format-Table` (display).

## 2.3 Users, Groups and NTFS Permissions

```
Get-LocalUser
Get-LocalGroup
Get-LocalGroupMember -Group 'Administrators'

$password = Read-Host "Password" -AsSecureString
New-LocalUser -Name "anna" -Password $password
Add-LocalGroupMember -Group 'Administrators' -Member 'anna'
Disable-LocalUser -Name 'anna'
Enable-LocalUser -Name 'anna'
```

Use standard accounts for daily work. Keep local Administrators membership small and review it regularly.

### NTFS permissions

| Permission     | Meaning                                               |
| -------------- | ----------------------------------------------------- |
| Full Control   | Read, write, delete, change permissions and ownership |
| Modify         | Read, write, execute and delete                       |
| Read & Execute | Read files and run programs                           |
| Read           | View contents                                         |
| Write          | Create or modify                                      |

```
icacls C:\Data
icacls C:\Data /grant anna:(R)
icacls C:\Data /grant anna:(M)
icacls C:\Data /grant anna:(OI)(CI)M /T
```

`OI` = inherit to files; `CI` = inherit to directories; `/T` = recursive. Share permissions and NTFS permissions both affect network-share access. Avoid broad `Everyone: Full Control` permissions.

## 2.4 Processes, Services and Scheduled Tasks

```
Get-Process
Get-Process -Name notepad
Stop-Process -Name notepad

Get-Service
Get-Service -Name Spooler
Restart-Service -Name Spooler

Get-CimInstance Win32_Service |
  Select-Object Name, State, StartMode, StartName, PathName

Get-ScheduledTask
```

CMD alternatives:

```
tasklist
taskkill /IM notepad.exe
sc query
sc qc Spooler
schtasks /query /fo LIST /v
```

Before stopping a process or service, check its purpose, dependencies and impact. For scheduled tasks, inspect the trigger, action, execution account and last result.

## 2.5 Event Viewer and Logs

Open `eventvwr.msc`.

| Log         | Typical content                       |
| ----------- | ------------------------------------- |
| System      | Drivers, services, boot and hardware  |
| Application | Application errors and crashes        |
| Security    | Logons, account and security activity |
| Setup       | Installation and update events        |

PowerShell:

```
Get-WinEvent -LogName System -MaxEvents 50
Get-WinEvent -LogName Application -MaxEvents 50

Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4624,4625
} -MaxEvents 30
```

Important Security Event IDs:

| ID          | Meaning                               |
| ----------- | ------------------------------------- |
| 4624        | Successful logon                      |
| 4625        | Failed logon                          |
| 4648        | Explicit credentials used             |
| 4672        | Special privileges assigned           |
| 4688        | Process created (if auditing enabled) |
| 4697        | Service installed                     |
| 4698        | Scheduled task created                |
| 4720        | User account created                  |
| 4728 / 4732 | Member added to a security group      |

Events depend on audit configuration and log retention. Missing events do not prove an action did not occur.

## 2.6 Storage, Networking and Repair

### Storage

```
Get-Disk
Get-Partition
Get-Volume
Get-Volume | Select-Object DriveLetter, FileSystem, Size, SizeRemaining
```

Open `diskmgmt.msc` for graphical disk management. Formatting, initializing or repartitioning can destroy data; verify the disk and backup first.

### Networking

```
ipconfig /all
ipconfig /flushdns
ipconfig /release
ipconfig /renew
ping <HOST>
tracert <HOST>
pathping <HOST>
nslookup example.com
netstat -ano
arp -a
```

```
Get-NetAdapter
Get-NetIPConfiguration
Get-NetRoute
Get-NetNeighbor
Resolve-DnsName example.com
Test-NetConnection <HOST> -Port <PORT>
Get-NetTCPConnection -State Listen
```

Basic diagnostic order

1. Check adapter status.
2. Check IP address, subnet, gateway and DNS.
3. Ping loopback and the default gateway.
4. Test an external IP.
5. Test DNS resolution.
6. Test the required TCP port.

A failed ping alone does not prove a host is offline; ICMP may be blocked. Avoid `ipconfig /release` over remote access unless you have an alternative connection.

### Windows repair and updates

Run in an elevated terminal:

```
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow
```

Check installed updates:

```
Get-HotFix
```

Use Windows Update (`ms-settings:windowsupdate`). Follow maintenance windows, plan restarts and verify services after patching.

## 2.7 Windows Firewall and Hardening

```
Get-NetFirewallProfile |
  Format-Table Name, Enabled, DefaultInboundAction

Get-NetFirewallRule -Enabled True |
  Select-Object DisplayName, Direction, Action
```

Open `wf.msc` to manage advanced firewall rules.

Windows hardening checklist

- Keep Windows and applications patched.
- Use standard accounts for daily work.
- Restrict local Administrators membership.
- Enable MFA where available.
- Keep Defender or approved endpoint protection active.
- Keep Windows Firewall enabled.
- Disable unnecessary services and remote access.
- Restrict RDP to trusted networks and approved users.
- Review privileged accounts and Security logs.
- Maintain tested backups.
- Follow organizational GPO/Intune policies.

## 2.8 Windows Quick Troubleshooting

| Symptom                | First checks                               |
| ---------------------- | ------------------------------------------ |
| Slow computer/high CPU | Task Manager, `resmon`, `Get-Process`      |
| Service unavailable    | `Get-Service`, Services, Event Viewer      |
| Application crashes    | Application log, Reliability Monitor       |
| No network             | `ipconfig /all`, `Get-NetAdapter`, gateway |
| IP works, name fails   | `Resolve-DnsName`, `ipconfig /flushdns`    |
| Port unavailable       | `Test-NetConnection <HOST> -Port <PORT>`   |
| Disk full              | `Get-Volume`, Disk Management              |
| System file corruption | DISM, then SFC                             |
| Suspicious login       | Security log, IDs 4624/4625                |

# 3. Windows Server and Active Directory

Scope: Windows Server, AD DS, domain accounts, Group Policy, DNS and DHCP.

## 3.1 Server Roles

| Role                      | Purpose                             |
| ------------------------- | ----------------------------------- |
| AD DS                     | Central identity and directory      |
| DNS                       | Name resolution                     |
| DHCP                      | Automatic IP configuration          |
| File and Storage Services | File shares and storage             |
| Hyper-V                   | Virtual machines                    |
| IIS                       | Web hosting                         |
| Remote Desktop Services   | Remote desktop/application delivery |

```
Get-WindowsFeature
Install-WindowsFeature -Name DNS -IncludeManagementTools
```

Installing AD DS is not the same as promoting a server to a domain controller.

## 3.2 Active Directory Concepts

AD DS centrally manages users, computers, groups and resources.

| Term                   | Meaning                                               |
| ---------------------- | ----------------------------------------------------- |
| Forest                 | Top-level AD structure                                |
| Domain                 | Directory and administrative boundary                 |
| OU                     | Organizes objects; scope for GPO and delegation       |
| Domain Controller (DC) | Hosts AD DS and authenticates users                   |
| Global Catalog         | Helps locate objects across the forest                |
| Site                   | Represents network topology and replication locations |
| Trust                  | Enables authentication across domains/forests         |

Typical structure:

```
Forest: company.local
└── Domain: company.local
    ├── OU: Users
    ├── OU: Computers
    ├── OU: Servers
    └── OU: Groups
```

## 3.3 Domain Controller Setup and Health

Basic deployment:

1. Install Windows Server.
2. Set a static IP and correct DNS.
3. Set the computer name.
4. Install AD DS.
5. Promote to a new or existing domain controller.
6. Verify DNS, authentication and replication.

```
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
```

Example lab deployment of a new forest:

```
Install-ADDSForest -DomainName "company.local"
```

Promotion requires planning, appropriate credentials and a restart. Do not run this on a production server without confirming the design.

Check DC health:

```
dcdiag
dcdiag /v
repadmin /replsummary
repadmin /showrepl
echo %LOGONSERVER%
```

## 3.4 Users and Groups

Management tools:

- `dsa.msc` — Active Directory Users and Computers.
- `gpmc.msc` — Group Policy Management.
- PowerShell ActiveDirectory module.

```
Import-Module ActiveDirectory
Get-ADUser -Filter *
Get-ADUser jsmith -Properties Enabled,LockedOut,LastLogonDate

New-ADUser `
  -Name "John Smith" `
  -SamAccountName "jsmith" `
  -UserPrincipalName "jsmith@company.local" `
  -Path "OU=Users,DC=company,DC=local"

Disable-ADAccount jsmith
Enable-ADAccount jsmith
Unlock-ADAccount jsmith
```

Use group-based access rather than assigning permissions individually.

| Group scope  | Typical use                                 |
| ------------ | ------------------------------------------- |
| Global       | Organize users from the same domain         |
| Domain Local | Assign permissions to resources in a domain |
| Universal    | Group members across domains in a forest    |

```
New-ADGroup -Name "IT-Support" -GroupScope Global `
  -GroupCategory Security -Path "OU=Groups,DC=company,DC=local"

Add-ADGroupMember -Identity "IT-Support" -Members "jsmith"
Get-ADGroupMember "IT-Support"
```

## 3.5 Group Policy

GPO centrally configures users and computers: password policy, firewall, updates, security settings, scripts and mapped drives.

GPOs can be linked to sites, domains and OUs. Typical processing order is LSDOU: Local → Site → Domain → OU (parent to child). Inheritance, link order and filtering can change the result.

```
gpupdate /force
gpresult /r
gpresult /h C:\Temp\gpo-report.html
```

If a GPO does not apply, check OU placement, security filtering, inheritance, replication and User vs Computer Configuration.

## 3.6 DNS and DHCP

### DNS

AD relies on DNS to locate domain controllers and services.

| Record   | Purpose                      |
| -------- | ---------------------------- |
| A / AAAA | Name to IPv4 / IPv6 address  |
| CNAME    | Alias                        |
| MX       | Mail server                  |
| PTR      | Reverse lookup               |
| SRV      | Locates services such as DCs |

```
Resolve-DnsName company.local
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.company.local
```

```
nslookup company.local
ipconfig /all
ipconfig /flushdns
```

Incorrect DNS is a common cause of domain join and authentication failures. Domain clients should use the organization's AD DNS servers.

### DHCP

DHCP supplies IP address, subnet mask, gateway, DNS and lease information.

DORA: Discover → Offer → Request → Acknowledge.

Check:

- DHCP service and authorization.
- Active scope and available addresses.
- Correct gateway and DNS options.
- DHCP relay/IP helper between VLANs.
- Client's network/VLAN.

DHCP can run on a separate server or network device; it does not have to run on a DC.

## 3.7 Kerberos, LDAP and Domain Join

- Kerberos: AD authentication using tickets; depends on DNS and accurate time.
- LDAP: Protocol for querying and managing directory information.

| Port        | Service  |
| ----------- | -------- |
| TCP/UDP 88  | Kerberos |
| TCP/UDP 389 | LDAP     |
| TCP 636     | LDAPS    |
| TCP/UDP 53  | DNS      |
| UDP 123     | NTP      |
| TCP 445     | SMB      |

Use secure LDAP transport where required. Do not expose directory services to untrusted networks.

Before joining a computer to a domain, verify DNS points to AD DNS, the DC is reachable, time is synchronized and authorized credentials are available.

```
Add-Computer -DomainName "company.local" -Credential "COMPANY\Administrator" -Restart

Get-CimInstance Win32_ComputerSystem |
  Select-Object Name, Domain, PartOfDomain
```

## 3.8 Replication and AD Troubleshooting

```
repadmin /replsummary
repadmin /showrepl
```

Force replication only when appropriate:

```
repadmin /syncall /AdeP
```

Do not use forced replication as a substitute for fixing DNS, connectivity, time or DC health.

| Symptom                       | First checks                                      |
| ----------------------------- | ------------------------------------------------- |
| Cannot join domain            | DNS, DC reachability, time, credentials           |
| User cannot log in            | Account enabled/locked, password, DC, time        |
| Account repeatedly locks      | Security logs, saved credentials, tasks, services |
| GPO not applying              | `gpresult`, OU, filtering, replication            |
| DC cannot be located          | DNS SRV records, DNS server, network              |
| Changes missing on another DC | `repadmin`, DNS, connectivity                     |
| Kerberos fails                | Time, DNS, account/SPN configuration              |
| Slow domain logon             | DNS, DC availability, GPO, network                |

Useful commands:

```
ipconfig /all
nslookup company.local
gpupdate /force
gpresult /r
dcdiag
repadmin /replsummary
```

```
Get-ADDomain
Get-ADForest
Get-ADDomainController -Filter *
Get-ADGroupMember "Domain Admins"
```

AD security checklist

- Use separate privileged accounts.
- Minimize Domain Admin membership.
- Enable MFA for administrative access where supported.
- Patch and restrict access to domain controllers.
- Monitor logons and privileged group changes.
- Review inactive accounts and stale computer objects.
- Maintain and test AD-aware backups.
- Monitor DNS, time and replication health.

# 4. Networking Fundamentals and Protocols

## 4.1 OSI and TCP/IP Models

| OSI layer | Name         | Function / examples                  |
| --------- | ------------ | ------------------------------------ |
| 7         | Application  | HTTP, DNS, SMTP, SSH                 |
| 6         | Presentation | Data format, encryption, compression |
| 5         | Session      | Session management                   |
| 4         | Transport    | TCP, UDP, ports                      |
| 3         | Network      | IPv4, IPv6, ICMP, routing            |
| 2         | Data Link    | Ethernet, Wi-Fi, VLAN, MAC           |
| 1         | Physical     | Copper, fiber, radio                 |

TCP/IP groups these into Application (OSI 5–7), Transport (4), Internet (3), Network Access (1–2).

Encapsulation:

`Application data → TCP segment/UDP datagram → IP packet → Ethernet frame → bits`

- IP addresses identify endpoints across networks.
- MAC addresses identify interfaces on the local network segment.
- Routers replace Layer 2 framing at each hop; IP addresses normally remain end-to-end unless NAT modifies them.

## 4.2 TCP, UDP, Ethernet and ARP

| TCP                                | UDP                            |
| ---------------------------------- | ------------------------------ |
| Connection-oriented                | Connectionless                 |
| Reliable, ordered delivery         | No built-in delivery guarantee |
| Acknowledgements/retransmissions   | Lower overhead                 |
| Common for web, SSH, file transfer | Common for DNS, voice/video    |

TCP handshake: `SYN → SYN-ACK → ACK`. `FIN` closes normally; `RST` resets a connection.

Ethernet is primarily Layer 2. A MAC address is normally 48 bits, shown in hexadecimal, e.g. `00:1A:2B:3C:4D:5E`.

ARP maps an IPv4 address to a MAC address on the local network. IPv6 uses Neighbor Discovery (NDP), not ARP.

## 4.3 IPv4 Addressing and Subnetting

IPv4 uses 32-bit addresses, usually written as four decimal octets (e.g. `192.168.1.10`).

An address and prefix identify the network and host portions:

- `/24` = `255.255.255.0`
- `/25` = `255.255.255.128`
- `/26` = `255.255.255.192`
- `/27` = `255.255.255.224`
- `/28` = `255.255.255.240`
- `/30` = `255.255.255.252`

For a typical IPv4 subnet, usable host addresses are between the network and broadcast addresses. Traditional usable-host count is `2^(32-prefix) - 2` (special cases such as `/31` and `/32` exist).

Example: `192.168.10.77/26`

- Mask: `255.255.255.192`
- Subnet range: `192.168.10.64–192.168.10.127`
- Network: `192.168.10.64`
- Broadcast: `192.168.10.127`
- Usable hosts: `192.168.10.65–192.168.10.126`

Important IPv4 ranges:

| Range            | Purpose          |
| ---------------- | ---------------- |
| `10.0.0.0/8`     | Private          |
| `172.16.0.0/12`  | Private          |
| `192.168.0.0/16` | Private          |
| `127.0.0.0/8`    | Loopback         |
| `169.254.0.0/16` | Link-local/APIPA |
| `0.0.0.0/0`      | Default route    |

## 4.4 IPv6 Basics

IPv6 uses 128-bit addresses, written in hexadecimal groups separated by colons.

- `::1` — loopback.
- `fe80::/10` — link-local.
- `fc00::/7` — unique local.
- `2000::/3` — global unicast.
- `::/0` — default route.

IPv6 uses NDP for neighbour discovery and supports SLAAC and DHCPv6 for address configuration. Do not assume IPv4 firewall rules protect IPv6 traffic; review both.

## 4.5 DNS, DHCP, NAT and Routing

- DNS: Resolves names to IP addresses. Common records: A, AAAA, CNAME, MX, PTR, SRV.
- DHCP: Automatically assigns IP configuration. DORA = Discover, Offer, Request, Acknowledge.
- NAT: Translates addresses. PAT/NAT overload lets multiple internal hosts share a public IP using ports. NAT is not a replacement for a firewall.
- Default gateway: Router/interface used to reach destinations outside the local subnet.
- Routing: Routers select the most specific matching route. Default routes are `0.0.0.0/0` (IPv4) and `::/0` (IPv6).

Useful commands:

```
ip addr
ip route
ip route get 8.8.8.8
```

```
ipconfig /all
route print
nslookup example.com
```

## 4.6 Important Network Ports

| Service            | Port    | Transport |
| ------------------ | ------- | --------- |
| FTP control        | 21      | TCP       |
| SSH / SFTP         | 22      | TCP       |
| Telnet             | 23      | TCP       |
| SMTP               | 25      | TCP       |
| DNS                | 53      | TCP/UDP   |
| DHCP server/client | 67/68   | UDP       |
| TFTP               | 69      | UDP       |
| HTTP               | 80      | TCP       |
| Kerberos           | 88      | TCP/UDP   |
| POP3               | 110     | TCP       |
| NTP                | 123     | UDP       |
| IMAP               | 143     | TCP       |
| SNMP               | 161/162 | UDP       |
| LDAP               | 389     | TCP/UDP   |
| HTTPS              | 443     | TCP       |
| SMB                | 445     | TCP       |
| SMTPS / submission | 465/587 | TCP       |
| LDAPS              | 636     | TCP       |
| IMAPS              | 993     | TCP       |
| POP3S              | 995     | TCP       |
| RDP                | 3389    | TCP/UDP   |

Prefer encrypted protocols (SSH, HTTPS, LDAPS) over Telnet, FTP and unencrypted management services.

````

**Part 2 of 2 — Sections 5–10**

```markdown
# 5. Switching, Routing and Network Infrastructure

## 5.1 Network Devices

| Device | Main function |
|---|---|
| Switch | Connects devices in a LAN; forwards frames using MAC addresses |
| Router | Connects different IP networks; forwards packets using routes |
| Layer 3 switch | Switching plus inter-VLAN routing |
| Firewall | Allows or blocks traffic according to security rules |
| Access Point | Connects wireless clients to a LAN |
| Modem/ONT | Connects the local network to the provider's access network |

## 5.2 VLANs and Switching

A **VLAN** separates a switched network into logical Layer 2 broadcast domains.

Benefits:
- Separates users, servers, guests and management.
- Reduces broadcast domains.
- Supports access control and network organisation.

### Access and trunk ports

- **Access port:** Carries traffic for one VLAN; commonly used for end devices.
- **Trunk port:** Carries multiple VLANs, usually using IEEE 802.1Q tags; commonly used between switches or to routers.

Cisco IOS examples:

```text
show vlan brief
show interfaces status
show interfaces trunk

configure terminal
vlan 10
 name USERS
interface gigabitEthernet 0/1
 switchport mode access
 switchport access vlan 10
````

Example trunk configuration (platform-dependent):

```
interface gigabitEthernet 0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

Verify the actual platform syntax and existing configuration before applying changes.

## 5.3 MAC Address Learning and STP

A switch learns source MAC addresses and the ports on which they arrive. It uses this MAC address table to forward frames.

```
show mac address-table
show mac address-table dynamic
```

Do not confuse:

- MAC table: MAC address → switch port.
- ARP table: IPv4 address → MAC address.

Spanning Tree Protocol (STP) prevents Layer 2 switching loops by blocking redundant paths while keeping them available as backups.

```
show spanning-tree
show spanning-tree vlan 10
```

Check the root bridge and port states when troubleshooting unexpected Layer 2 paths. Avoid changing STP settings without understanding the topology; a loop can disrupt the network.

## 5.4 Inter-VLAN Routing

Devices in different VLANs need Layer 3 routing to communicate.

Common methods:

1. Router-on-a-stick: One router interface uses multiple VLAN subinterfaces over a trunk.
2. Layer 3 switch: Switched Virtual Interfaces (SVIs) provide gateways for VLANs.

Example SVI configuration (platform-dependent):

```
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```

Verify VLAN membership, trunk allowance, gateway configuration, routing and ACL/firewall rules.

## 5.5 Routing

Routers select a route based on the destination IP and the most specific matching prefix.

### Static routes

Manually configured; useful for small or predictable networks.

```
show ip route
ip route 192.168.20.0 255.255.255.0 192.168.1.2
```

### Dynamic routing

Routers exchange routing information automatically.

Common protocols:

- OSPF: Link-state routing protocol used within an organisation or autonomous system.
- BGP: Path-vector protocol used between autonomous systems and widely on the internet.
- RIP: Older distance-vector protocol; limited scalability.

Useful checks:

```
show ip route
show ip protocols
show ip interface brief
```

Do not introduce or change routing protocols without understanding route advertisements, metrics, authentication and network impact.

## 5.6 EtherChannel and Physical Infrastructure

EtherChannel combines multiple physical links into one logical link for increased capacity and redundancy. LACP is a common negotiation protocol.

```
show etherchannel summary
show interfaces port-channel
```

Members should have compatible speed, duplex, VLAN and trunk settings.

### Copper, fiber, SFP and PoE

- Copper Ethernet: Common for endpoint connections; observe cable category and distance limits.
- Fiber: Used for longer distances and higher-speed links; single-mode and multimode fiber have different optics and distance characteristics.
- SFP/SFP+: Pluggable transceivers; verify speed, fiber type, wavelength and compatibility at both ends.
- PoE: Supplies power over Ethernet to devices such as access points, phones and cameras. Check switch PoE budget and device requirements.

When a link is down, check power, cable, transceiver, link LEDs, port state, speed/duplex and interface errors.

## 5.7 Essential Cisco IOS Verification

```
show running-config
show startup-config
show interfaces status
show interfaces
show ip interface brief
show vlan brief
show interfaces trunk
show mac address-table
show spanning-tree
show ip route
show etherchannel summary
show logging
```

### Safe network change procedure

1. Confirm authorisation, objective and affected devices.
2. Save/export the current configuration.
3. Check dependencies and expected impact.
4. Prepare a rollback plan.
5. Apply one controlled change.
6. Verify interfaces, VLANs, routes and connectivity.
7. Check logs and monitoring.
8. Save the configuration only after successful verification.

# 6. Network and System Troubleshooting

## 6.1 General Method

Use a consistent process:

1. Identify: What is failing, where, when and for whom?
2. Scope: One user, one device, one VLAN or the whole environment?
3. Check recent changes: Configuration, updates, cabling, firewall or routing.
4. Inspect: Collect command output, logs and monitoring data.
5. Isolate: Test one layer or component at a time.
6. Fix: Make the smallest authorised and reversible change.
7. Verify: Test the original function and check for new errors.
8. Document: Record evidence, cause, fix and rollback.

Do not begin by rebooting devices, disabling firewalls or flushing configurations.

## 6.2 Network Troubleshooting Sequence

Follow the path from the local device toward the destination.

### 1. Physical link and adapter

Check power, cable, Wi-Fi association, link LEDs, adapter state and switch port.

```
ip -br link
ip -br addr
```

```
Get-NetAdapter
Get-NetIPConfiguration
```

### 2. IP configuration

Check address, subnet mask/prefix, gateway and DNS.

```
ip addr
ip route
```

```
ipconfig /all
route print
```

Look for:

- Missing or incorrect IP address.
- Unexpected `169.254.x.x` IPv4 address (often indicates DHCP failure).
- Incorrect subnet or gateway.
- Incorrect DNS server.
- Duplicate IP address.

### 3. Gateway and local connectivity

```
ping -c 4 <GATEWAY>
```

```
ping <GATEWAY>
```

If the gateway is unreachable, check local VLAN, Wi-Fi, cabling, switch port, IP settings and gateway availability.

### 4. Routing

```
ip route
ip route get <DESTINATION>
```

```
route print
tracert <DESTINATION>
```

Check the default route, destination route, next hop and return path.

### 5. DNS

If an IP address works but a hostname does not, investigate DNS.

```
getent hosts example.com
resolvectl status
```

```
Resolve-DnsName example.com
```

```
nslookup example.com
ipconfig /flushdns
```

Verify the configured DNS server and whether it can resolve the required name.

### 6. Service port

A host can respond to ping while the required application port remains unavailable.

```
Test-NetConnection <HOST> -Port <PORT>
```

```
ss -lntup
```

Check whether the service is listening, whether the host firewall allows it and whether network firewalls/ACLs block the path.

### 7. Trace the path and inspect packets

```
tracert <HOST>
pathping <HOST>
```

```
tracepath <HOST>
sudo tcpdump -ni <INTERFACE> host <IP>
```

Use packet capture only when needed and only on networks you are authorised to inspect. Captures may contain sensitive information.

### 8. Check logs

Inspect endpoint, server, firewall, switch and router logs around the time of the failure.

## 6.3 Common Network Symptoms

| Symptom                       | Likely areas to check                                |
| ----------------------------- | ---------------------------------------------------- |
| No network access             | Link, adapter, VLAN, DHCP, IP settings               |
| `169.254.x.x` address         | DHCP server, scope, relay, VLAN                      |
| Gateway unreachable           | Local link, VLAN, subnet, gateway                    |
| IP works, hostname fails      | DNS client/server, records, resolver                 |
| One port unavailable          | Service state, host firewall, ACL, routing           |
| Some VLANs cannot communicate | Inter-VLAN routing, ACLs, gateway                    |
| Intermittent connection       | Packet loss, duplex, cable, congestion, Wi-Fi        |
| Only one host affected        | Local configuration, adapter, endpoint firewall      |
| Multiple hosts affected       | Shared switch, router, DNS, DHCP or upstream service |

A failed ping does not prove a host is offline; ICMP may be filtered.

## 6.4 System Troubleshooting

### High CPU

Linux:

```
top
ps aux --sort=-%cpu | head
```

Windows:

```
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
```

Check the process, recent changes, workload and whether usage is sustained.

### High memory usage

Linux:

```
free -h
top
```

Windows:

```
Get-Process |
  Sort-Object WorkingSet64 -Descending |
  Select-Object -First 10
```

Check memory pressure, unusual processes and possible leaks.

### Disk space problems

Linux:

```
df -h
du -sh /var/log/*
lsblk
```

Windows:

```
Get-Volume
Get-PSDrive -PSProvider FileSystem
```

Check free space, large logs, temporary files, application data and disk health. Do not delete logs or application data without checking retention and ownership.

### Service problems

Linux:

```
systemctl status <SERVICE>
journalctl -u <SERVICE> --since today
```

Windows:

```
Get-Service
Get-Service -Name <SERVICE>
```

Check service state, startup type, dependencies, permissions and related logs before restarting.

## 6.5 Safe Troubleshooting Checklist

- Confirm the affected device, user, service and time.
- Determine whether other devices have the same issue.
- Record current IP, route and adapter configuration.
- Check recent changes and relevant logs.
- Confirm authorisation.
- Back up configuration before editing.
- Change one thing at a time.
- Verify the result.
- Document the fix and rollback method.

# 7. Cybersecurity Fundamentals and System Hardening

## 7.1 Core Principles

| Principle        | Practical meaning                                                     |
| ---------------- | --------------------------------------------------------------------- |
| Confidentiality  | Only authorised people access information                             |
| Integrity        | Prevent unauthorised changes                                          |
| Availability     | Keep systems and data accessible                                      |
| Least privilege  | Grant only required permissions                                       |
| Defense in depth | Use multiple independent controls                                     |
| Zero Trust       | Verify access explicitly; do not trust based only on network location |

Common threats:

- Malware: Ransomware, spyware, trojans and other malicious software.
- Phishing: Deceptive messages designed to steal credentials or deliver malware.
- Credential attacks: Password spraying, brute force and stolen credentials.
- Privilege escalation: Obtaining permissions beyond those authorised.
- Lateral movement: Moving from one compromised system to another.
- Misconfiguration: Unnecessary exposure, weak permissions or insecure defaults.
- Denial of Service: Disrupting service availability.

## 7.2 Identity and Access Management

- Use individual accounts; avoid shared administrator accounts.
- Use MFA, especially for privileged and remote access.
- Use standard accounts for everyday work.
- Restrict and regularly review administrator/group membership.
- Disable or remove accounts when no longer required.
- Use strong, unique passwords and an approved password manager.
- Review service accounts, credentials and permissions.
- Apply least privilege to users, services and scheduled tasks.

## 7.3 Linux Hardening

### Updates and accounts

```
sudo apt update && sudo apt upgrade
getent passwd
sudo -l
```

Remove unnecessary software and accounts. Review privileged access.

### SSH

- Prefer key-based authentication where appropriate.
- Restrict access to authorised users and management networks.
- Avoid direct root login.
- Keep OpenSSH updated.
- Validate configuration before reloading:

```
sudo sshd -t
sudo systemctl reload ssh
```

Keep an existing remote session open until a second connection is verified.

### Permissions and firewall

- Avoid `chmod 777`.
- Review ownership, permissions and ACLs.
- Enable only required firewall access.
- Monitor authentication and system logs.

```
sudo ufw status verbose
sudo nft list ruleset
```

## 7.4 Windows Hardening

- Keep Windows and applications patched.
- Keep Microsoft Defender or approved endpoint protection enabled.
- Keep Windows Firewall enabled.
- Restrict local Administrators membership.
- Use MFA for privileged and remote access.
- Disable unnecessary services and remote access.
- Restrict RDP to trusted networks and approved users.
- Apply security policy through approved GPO/Intune configuration.
- Monitor Security and System logs.

Useful checks:

```
Get-MpComputerStatus
Get-NetFirewallProfile
Get-NetFirewallRule -Enabled True
```

## 7.5 Patching and Vulnerability Management

Practical process:

1. Maintain an inventory of systems and software.
2. Identify missing security updates and known vulnerabilities.
3. Prioritise internet-facing, critical and actively exploited systems.
4. Test updates where appropriate.
5. Deploy during approved maintenance windows.
6. Plan restarts and rollback.
7. Verify patch installation and service health.
8. Track exceptions and unresolved vulnerabilities.

## 7.6 Logging, Integrity and Backups

### Logging

Collect and review relevant:

- Authentication and account events.
- Privileged actions.
- Service and application errors.
- Firewall and network device events.
- Endpoint security alerts.
- Configuration changes.

Protect logs from unauthorised modification and retain them according to policy.

### File integrity

Use hashes to compare files and detect changes:

```
sha256sum <FILE>
```

```
Get-FileHash <FILE> -Algorithm SHA256
```

A hash change indicates different file content; it does not by itself prove malicious activity.

### Backups — 3-2-1 rule

- Keep 3 copies of important data.
- Use 2 different storage types.
- Keep 1 copy off-site or otherwise isolated.

Protect backups from ransomware and unauthorised access. Test restoration regularly; a successful backup job does not prove that recovery will work.

## 7.7 System Hardening Checklist

- Patch operating systems and applications.
- Remove unnecessary accounts, services and software.
- Apply least privilege and MFA.
- Restrict remote administration.
- Enable host firewalls and endpoint protection.
- Review privileged groups and permissions.
- Centralise and protect logs.
- Maintain isolated backups and test restoration.
- Document exceptions and configuration changes.

# 8. Network Security

## 8.1 Firewalls and Traffic Filtering

A firewall permits or blocks traffic according to rules based on factors such as source, destination, protocol, port and connection state.

Common types:

- Stateless packet filtering: Evaluates individual packets.
- Stateful firewall: Tracks connection state.
- Next-generation firewall (NGFW): May include application awareness, intrusion prevention and other controls.
- Host firewall: Protects an individual endpoint or server.

Practical principles:

- Deny unsolicited inbound traffic by default where appropriate.
- Allow only required services and ports.
- Restrict management access to trusted networks.
- Review rules and remove obsolete exceptions.
- Log relevant denied or suspicious traffic.

Useful checks:

```
sudo ufw status verbose
sudo nft list ruleset
```

```
Get-NetFirewallProfile
Get-NetFirewallRule -Enabled True
```

## 8.2 Network Segmentation

Separate systems by function and trust level, for example:

- User workstations.
- Servers.
- Guest Wi-Fi.
- Management interfaces.
- Security infrastructure.
- Critical systems.

Use VLANs for Layer 2 separation and ACLs/firewalls to control traffic between segments. A VLAN alone does not prevent all communication if routing permits it.

## 8.3 Secure Remote Administration

### SSH

- Use keys where appropriate.
- Restrict source networks and authorised users.
- Avoid direct root login.
- Keep the server patched.
- Monitor authentication.

### RDP

- Do not expose RDP directly to the public internet.
- Use VPN or an approved remote-access gateway.
- Require MFA where supported.
- Restrict users and source networks.
- Keep systems patched and monitor logons.

### VPN

A VPN creates an encrypted connection over an untrusted network. Use approved protocols and clients, MFA and access restrictions. Grant access only to the networks and services required.

## 8.4 Common Network Security Controls

| Control                | Purpose                                          |
| ---------------------- | ------------------------------------------------ |
| ACL                    | Permit or deny traffic based on rules            |
| IDS                    | Detect and alert on suspicious activity          |
| IPS                    | Detect and block selected malicious traffic      |
| DHCP snooping          | Help prevent rogue DHCP servers                  |
| Dynamic ARP Inspection | Help prevent ARP spoofing                        |
| Port security          | Restrict MAC addresses on switch ports           |
| 802.1X                 | Authenticate devices/users before network access |
| SNMPv3                 | Secure network-device monitoring                 |
| DNS filtering          | Block or alert on selected domains               |
| Egress filtering       | Restrict outbound connections                    |

Use controls supported by the infrastructure and organisation's security policy.

## 8.5 Network Security Checklist

- Separate user, server, guest and management networks.
- Restrict inter-VLAN traffic with ACLs or firewalls.
- Disable unused switch ports.
- Use secure management protocols.
- Restrict SSH, RDP and network-device administration.
- Use MFA for remote access.
- Prefer SNMPv3.
- Monitor firewall and network-device logs.
- Review firewall rules periodically.
- Update network-device firmware.
- Back up device configurations.

# 9. SOC, Security Monitoring and Incident Response

## 9.1 SOC Fundamentals

A Security Operations Center (SOC) monitors systems and networks, investigates security alerts and coordinates incident response.

Typical responsibilities:

- Monitor SIEM, EDR and security alerts.
- Investigate suspicious activity.
- Identify affected users, devices and systems.
- Correlate endpoint, network, identity and cloud events.
- Escalate incidents.
- Preserve evidence and document actions.

| Term           | Meaning                                                          |
| -------------- | ---------------------------------------------------------------- |
| SIEM           | Collects and correlates logs and security events                 |
| EDR            | Detects and investigates endpoint activity; may support response |
| IOC            | Indicator of compromise, e.g. malicious IP, domain or hash       |
| IOA            | Indicator of attack, based on suspicious behaviour               |
| TTP            | Tactics, techniques and procedures used by an attacker           |
| False positive | Alert that is not a real security incident                       |
| False negative | Malicious activity that was not detected                         |

An IOC is a lead for investigation, not proof by itself.

## 9.2 Alert Triage

For each alert, establish:

1. Alert details: Rule, severity, time, source and detection reason.
2. Affected asset: Hostname, IP, user, owner and business importance.
3. Process context: Process name, path, command line, hash, parent and child processes.
4. User context: Account, privileges, logon type and whether the activity is expected.
5. Persistence: New services, scheduled tasks, startup entries or registry changes.
6. Network activity: Remote IPs/domains, ports, DNS queries and unusual connections.
7. Scope: Other affected hosts, accounts or related alerts.
8. Decision: Benign, suspicious, confirmed incident or needs escalation.

Do not decide based only on a process name or file name. Check path, signature, hash, parent process, command line and behaviour.

## 9.3 Important Windows Security Events

| Event ID    | Meaning                                |
| ----------- | -------------------------------------- |
| 4624        | Successful logon                       |
| 4625        | Failed logon                           |
| 4648        | Explicit credentials used              |
| 4672        | Special privileges assigned            |
| 4688        | Process creation (if auditing enabled) |
| 4697 / 7045 | Service installed                      |
| 4698        | Scheduled task created                 |
| 4720        | User account created                   |
| 4728        | Member added to global security group  |
| 4732        | Member added to local security group   |

PowerShell search:

```
Get-WinEvent -FilterHashtable @{
  LogName = 'Security'
  Id = 4624,4625,4688,4697,4698,4720,4728,4732
} -MaxEvents 100
```

Event availability depends on audit configuration and log retention. Check the event's account, timestamp, logon type, source address and surrounding activity.

## 9.4 Sysmon and Investigation Tools

Sysmon provides additional Windows telemetry when installed and configured. Event coverage depends on its configuration.

Useful Sysmon Event IDs:

| ID    | Activity            |
| ----- | ------------------- |
| 1     | Process creation    |
| 3     | Network connection  |
| 7     | Image/module loaded |
| 10    | Process access      |
| 11    | File created        |
| 12–14 | Registry events     |
| 22    | DNS query           |

Useful tools:

| Tool                            | Purpose                                  |
| ------------------------------- | ---------------------------------------- |
| Task Manager / Process Explorer | Process tree and process details         |
| Autoruns                        | Startup and persistence locations        |
| TCPView                         | Processes and network connections        |
| Sysmon                          | Additional endpoint telemetry            |
| Defender / EDR                  | Threat detection and investigation       |
| Wireshark / tcpdump             | Packet analysis                          |
| SIEM                            | Centralised event search and correlation |

## 9.5 MITRE ATT&CK

MITRE ATT&CK is a knowledge base of adversary tactics and techniques. It helps describe observed behaviour and structure investigations.

Examples:

| Technique                         | ID        | Example                                          |
| --------------------------------- | --------- | ------------------------------------------------ |
| Command and Scripting Interpreter | T1059     | Shells and scripts                               |
| Scheduled Task                    | T1053.005 | Persistence or execution through scheduled tasks |
| Windows Service                   | T1543.003 | Create or modify a service                       |
| Registry Run Keys / Startup       | T1547.001 | Persistence through startup locations            |
| OS Credential Dumping             | T1003     | Attempts to obtain credentials                   |
| Remote Services                   | T1021     | Remote access/lateral movement                   |
| Impair Defenses                   | T1562.001 | Disable or weaken security tools                 |

A technique mapping alone is not proof of malicious activity; assess the full context.

## 9.6 Incident Response

Follow the organisation's incident response plan.

1. Identify: Confirm the alert and affected systems/accounts.
2. Preserve evidence: Record timestamps, logs, hashes, command lines, processes and connections.
3. Assess impact: Determine affected hosts, users, services and business criticality.
4. Contain: Use approved actions, such as endpoint isolation, blocking an IOC or revoking sessions.
5. Escalate: Notify the incident response team and relevant stakeholders.
6. Eradicate and recover: Remove the cause, restore systems safely and verify integrity.
7. Monitor: Look for persistence, lateral movement and recurring activity.
8. Document: Record evidence, decisions, actions and lessons learned.

Do not:

- Delete suspicious files before evidence is collected, unless immediate containment requires it.
- Clear event logs.
- Disable EDR or antivirus to make an alert disappear.
- Run suspected malware.
- Reset passwords without considering active sessions, tokens and incident procedures.
- Isolate critical production systems without considering impact and authorisation.

Remember: Observe → Preserve context → Verify → Assess impact → Escalate or take approved action.

# 10. Essential Administration Procedures and Reference

## 10.1 Before a Change

- Confirm the request and authorisation.
- Identify affected systems, users and services.
- Check dependencies and expected impact.
- Record the current configuration.
- Back up relevant data/configuration.
- Prepare a rollback procedure.
- Schedule a maintenance window if required.
- Notify affected users or teams.

## 10.2 After a Change

- Verify the configuration.
- Test the affected service.
- Confirm dependent services still work.
- Check logs and monitoring.
- Confirm security controls remain enabled.
- Save the configuration if appropriate.
- Document the result and outstanding issues.

## 10.3 Cross-Platform Command Reference

| Task             | Linux                 | Windows                                 |
| ---------------- | --------------------- | --------------------------------------- |
| Current user     | `whoami`              | `whoami`                                |
| Hostname         | `hostname`            | `hostname`                              |
| IP configuration | `ip addr`             | `ipconfig /all`                         |
| Routing table    | `ip route`            | `route print`                           |
| DNS lookup       | `getent hosts <NAME>` | `nslookup <NAME>`                       |
| Connectivity     | `ping <HOST>`         | `ping <HOST>`                           |
| Trace route      | `tracepath <HOST>`    | `tracert <HOST>`                        |
| Listening ports  | `ss -lntup`           | `netstat -ano`                          |
| Processes        | `ps aux`              | `Get-Process`                           |
| Services         | `systemctl`           | `Get-Service`                           |
| System logs      | `journalctl`          | Event Viewer / `Get-WinEvent`           |
| Disk space       | `df -h`               | `Get-Volume`                            |
| File hash        | `sha256sum <FILE>`    | `Get-FileHash <FILE> -Algorithm SHA256` |

## 10.4 Incident and Change Documentation

Record for every significant incident or change:

| Field            | Information                           |
| ---------------- | ------------------------------------- |
| Date/time        | Include timezone                      |
| System           | Hostname, IP and environment          |
| Request/incident | Ticket or incident reference          |
| Description      | What happened or changed              |
| Evidence         | Logs, commands, screenshots or hashes |
| Action           | What was done and by whom             |
| Result           | Whether the issue was resolved        |
| Rollback         | How to restore the previous state     |
| Follow-up        | Remaining risks or required actions   |

## 10.5 Final Operational Rules

1. Diagnose before changing configuration.
2. Use least privilege.
3. Do not expose management interfaces unnecessarily.
4. Keep systems patched and monitored.
5. Back up configurations and important data.
6. Test restoration, not just backup creation.
7. Preserve evidence during security incidents.
8. Document changes and verify results.
9. Follow organisational procedures and escalation paths.
10. Understand commands before running them with elevated privileges.

Core principle: Make controlled, authorised and reversible changes. Verify the result and leave a clear record for the next administrator.
