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
#1.Check Btrfs
sudo btrfs subvolume list /

#2. Create and mount temporary backup mount point (it's needed to avoid issues related to btrfs automatically not putting subvolume in the correct level) 
sudo mkdir -p /mnt/btrfs-top
sudo mount -o subvolid=5 /dev/nvme0n1p2 /mnt/btrfs-top

#3. Create subvolume
sudo btrfs subvolume create /mnt/btrfs-top/@backup

#4. Check if subvolume was created
sudo btrfs subvolume list /

#5. Unmount from temporary mount point
sudo umount /mnt/btrfs-top

#6. Create the final backup mount point
sudo mkdir /backup

#7. Mount the subvolume
sudo mount -o subvol=/@backup /dev/nvme0n1p2 /backup

#8. Test if directory works properly (create a test file)
sudo touch /backup/test.txt

#9. Backup fstab
sudo cp /etc/fstab /etc/fstab.backup

#10. Add backup subvolume to fstab, to boot it automatically
UUID=... /backup btrfs subvol=/@backup,defaults 0 0

#11. Test if subvolume mounts automatically
sudo umount /backup
sudo mount -a
findmnt /backup
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
