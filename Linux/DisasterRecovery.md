# 9. Disaster Recovery

## Used technologies

* `Clonezilla`
* `btrfs`
* `tar`
* `rsync`

## Tasks

### 1. Create a local backup of the most important directories, such as `/home` and `/etc`.
---
#### 1a. Create a new partition on a different drive for storing backups.

```bash
# Place for future command
```

#### 1b. Create a Btrfs subvolume on the current drive for storing backups.

```bash
sudo btrfs subvolume create /@backup
```

```bash
# Place for future command
```

---

### 2. Back up information about downloaded packages and package repositories.

```bash
# Place for future command
```

---

### 3. Back up information about installed disks and storage devices.

```bash
# Place for future command
```

---

### 4. Back up information about installed browser extensions.

```bash
# Place for future command
```

---

### 5. Back up browser bookmarks.

```bash
# Place for future command
```

---

### 6. Archive the most important files.

```bash
# Place for future command
```

---

### 7. Create a complete disk image backup on an external device using Clonezilla.

```bash
# Place for future command
```

---

### 8. Verify that your Bitwarden master password is strong and that you can remember it.

```bash
# Place for future command
```
