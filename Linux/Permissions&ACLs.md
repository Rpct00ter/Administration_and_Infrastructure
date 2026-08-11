# Permissions and ACL Management
## Used commands

* `chmod`
* `chown`
* `chgrp`
* `setfacl`
* `getfacl`
* `umask`
* `visudo`

# Tasks

## 1. Grant the owner full permissions, the group read and write permissions, and deny all permissions to other users for the `data.txt` file.

```bash
sudo chmod 760 data.txt
```

## 2. Change the owner of `backup.tar` to the 'admin' user and the group to 'backup`.

```bash
sudo chown admin:backup backup.tar
```

## 3. Add the john user to the developers group.

```bash
sudo usermod -aG developers john
```

## 4. Grant the john user read permission to the specified file using an ACL.

```bash
sudo setfacl -m u:john:r data.txt
```

## 5. Remove all ACL entries from the specified file.

```bash
sudo setfacl -b data.txt
```

## 6. Display the ACL entries for the specified file.

```bash
getfacl data.txt
```

## 7. Check the current `umask` value.

```bash
umask
```
> **Note:** 'umask' controls which permissions are automatically removed when new files and directories are created.
## 8. Set a new `umask` value for the current session.

```bash
umask 077
```


## 9. Display the permissions of the specified file.

```bash
ls -al data.txt
```

## 10. Find all files with the Sticky Bit set.

```bash
sudo find / -type f -perm -1000
```
> **Note:** If Sticky Bit is set, that means the users can normally delete only their own files.
> 
## 11. Find all files with the SUID bit set.

```bash
sudo find / -type f -perm -4000 2>/dev/null
```
> **Note:** If SUID is set, the program runs with the permissions of the file's owner instead of mine.

## 12. Find all files with the SGID bit set.

```bash
sudo find / -type f -perm -2000 2>/dev/null
```
> **Note:** If a directory has SGID set, newly created files inherit the directory's group.

## 13. Grant the developer user permission to execute commands using `sudo`.

```bash
sudo usermod -aG sudo developer
```

## 14. Remove the developer user's permission to execute commands using `sudo`.

```bash
sudo gpasswd -d developer sudo
```

## 15. Using 'sudoers' grant the developer user permission to use 'sudo', but only to change ip addresses.
```bash
#Adds and opens new modular ruleset for 'developer' user
sudo visudo -f /etc/sudoers.d/developer
```
Line that adds custom sudo rule enabling developer to change ip addresses:
```bash
developer ALL=(root) /usr/sbin/ip addr add *, /usr/sbin/ip addr del *
```
