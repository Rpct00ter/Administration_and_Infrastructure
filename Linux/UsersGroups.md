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
groupadd developers

useradd -m john

usermod -aG developers john
usermod -aG developers developer
```
## 4. Show all users
```bash
cat /etc/passwd
```
## 5. Show all groups
```bash
cat /etc/group
```
## 6. Lock the 'john' user
```bash
# Place for future command
```
## 7. Check if 'john' user is locked
```bash
  sudo passwd -S john
```
## 8. Delete the 'john' user
```bash
# Place for future command
```
