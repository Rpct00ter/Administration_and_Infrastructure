# Permissions and ACL Management
## Used commands

* `chmod`
* `chown`
* `chgrp`
* `setfacl`
* `getfacl`
* `umask`

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

## 8. Set a new `umask` value for the current session.

```bash
# Place for future command
```

## 9. Create a new file and check which permissions were assigned to it based on the current `umask`.

```bash
# Place for future command
```

## 10. Display the permissions of the specified file.

```bash
# Place for future command
```

## 11. Find all files with the Sticky Bit set.

```bash
# Place for future command
```

## 12. Find all files with the SUID bit set.

```bash
# Place for future command
```

## 13. Find all files with the SGID bit set.

```bash
# Place for future command
```

## 14. Grant the developer user permission to execute commands using `sudo`.

```bash
# Place for future command
```

## 15. Remove the developer user's permission to execute commands using `sudo`.

```bash
# Place for future command
```

