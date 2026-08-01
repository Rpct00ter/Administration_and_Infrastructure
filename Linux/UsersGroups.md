# Users and Groups Management

## Used commands
- `useradd`
- `passwd`
- `usermod`

# Tasks

## 1. Create a user named `developer` with a home directory.
```bash
  useradd -m developer
```

## 2. Set or change a password for the `developer` user.
```bash
  sudo passwd developer
```

## 3. Create the 'developers' group. Create user 'john' and add `john` and 'developer' user to the `developers` group.
```bash
  sudo groupadd developers

  sudo useradd -m john

  sudo usermod -aG developers john
  sudo usermod -aG developers developer
```
## 4. Show all users
```bash
  sudo cat /etc/passwd
```
## 5. Show all groups
```bash
  sudo cat /etc/group
```
## 6. Switch to the 'john' user and start a full login shell. Verify who you are logged as. Then come back to default user.
```bash
  su - john

  whoami

  exit
```
## 7. Log in as a 'john' user without knowing his password (sudo privileges required).
```bash
  sudo -u john -i
```

## 8. Lock the 'john' user
```bash
  sudo passwd -l john
```
## 9. Check if 'john' user is locked
```bash
  sudo passwd -S john
```
## 10. Unlock the 'john' user
```bash
  sudo passwd -u john
```
## 11. Delete the 'john' user with his home directory
```bash
  sudo userdel -r john
```
> **Note:** Removing "-r" will delete the user but keep his home directory content
