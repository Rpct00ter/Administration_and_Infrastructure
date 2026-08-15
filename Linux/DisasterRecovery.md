# 9. Disaster Recovery

## Used technologies

* `Clonezilla`
* `btrfs`
* `tar`
* `rsync`

## Tasks

### 1. Create a stable place to store backups
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

---
### 2. Backup the most important directories (like 'etc' and 'home') to the backup place. 
```bash
#1. Create backup directories
sudo mkdir -p /backup/home /backup/etc

#2. Backup /home
sudo rsync -aAXHv /home/ /backup/home/

#3. Backup /etc
sudo rsync -aAXHv /etc/ /backup/etc/

#4. Check backup sizes
sudo du -sh /home /backup/home
sudo du -sh /etc /backup/etc

#5. Test what would be copied without actually copying anything
sudo rsync -aAXHv --dry-run /home/ /backup/home/
sudo rsync -aAXHv --dry-run /etc/ /backup/etc/
```
---
### 3. Back up information about downloaded packages and package repositories.

```bash
#1. Create package backup directory
sudo mkdir -p /backup/packages

#2. Backup information about all installed packages
dpkg-query -W -f='${binary:Package}\t${Version}\n' | sudo tee /backup/packages/installed-packages.txt > /dev/null

#3. Backup manually installed packages
apt-mark showmanual | sudo tee /backup/packages/manually-installed.txt > /dev/null

#4. Backup APT repository configuration
sudo cp -a /etc/apt /backup/packages/apt-repositories

#5. Create a readable list of configured repositories
grep -RhsE '^[[:space:]]*(deb|deb-src)[[:space:]]' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null | sudo tee /backup/packages/repositories.txt > /dev/null

#6. Backup downloaded .deb packages currently stored in APT cache
sudo mkdir -p /backup/packages/downloaded-packages
sudo rsync -a /var/cache/apt/archives/ /backup/packages/downloaded-packages/

#7. Check the backup
find /backup/packages -maxdepth 2 -type f -print
```

---

### 4. Back up information about installed disks and storage devices.

```bash
#1. Backup information about disks and partitions
sudo fdisk -l | sudo tee /backup/system-info/disks.txt > /dev/null

#2. Backup UUID and filesystem information
sudo blkid | sudo tee /backup/system-info/blkid.txt > /dev/null
```

---

### 5. Back up information about installed browser extensions.

```bash
#1. Create browser backup directory
sudo mkdir -p /backup/browser

#2. Find extension directory and copy it's content to the backup (I use firefox)
jq -r '.addons[] | select(.type == "extension") | "\(.defaultLocale.name) — \(.version) — \(.id)"' \
~/.mozilla/firefox/*/extensions.json \
| sudo tee /backup/browser/firefox-extensions.txt > /dev/null
```

---

### 6. Back up browser bookmarks.
>**Note**: This step is fairly easy. I'm pretty sure that all of the common browsers have the option to export and save all the bookmarks into one file. Search through the menu and you shall find it. Save the extracted file to the '/backup' directory.
---

### 7. Archive the most important files.

```bash
# Place for future command
```
---

### 8. Create a complete disk image backup on an external device using Clonezilla.

```bash
# Place for future command
```

---
### 9. Verify that your Bitwarden master password is strong and that you can remember it.

```bash
# Place for future command
```
