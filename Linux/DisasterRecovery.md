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
#1. Check available disks and partitions
lsblk -f

#2. Open the partitioning tool for the new disk (replace /dev/sdX with the correct disk) and create a new partition.
sudo fdisk /dev/sdX

#3. Check if the new partition was created
lsblk

#4. Create an ext4 filesystem on the new partition (deletes existing data on the selected partition)
sudo mkfs.ext4 /dev/sdX1

#5. Create the backup mount point
sudo mkdir /backup

#6. Temporarily mount the new partition
sudo mount /dev/sdX1 /backup

#7. Check if the partition is mounted correctly
findmnt /backup

#8. Get the UUID of the new partition
sudo blkid /dev/sdX1

#9. Backup fstab before modifying it
sudo cp /etc/fstab /etc/fstab.backup

#10. Add the new partition to fstab so it mounts automatically at boot
UUID=... /backup ext4 defaults 0 2

#11. Test if the partition mounts automatically
sudo umount /backup
sudo mount -a
findmnt /backup
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
