# File Systems and Disks

## Used commands

* `df`
* `du`
* `lsblk`
* `blkid`
* `mount`
* `umount`
* `mkfs`
* `e2label`
* `fdisk`
* `parted`

## Tasks

### 1. Display disk space usage for the `/home` directory.
```bash
sudo du -h /home
```

### 2. Display the disk usage of all mounted file systems.
```bash
df -h
```

### 3. Display the UUIDs of all block devices.
```bash
blkid
```


### 4. Check inode usage on all file systems.
```bash
df -i
```


### 5. Create a new partition on the `/dev/sdb` disk.
```bash
sudo fdisk /dev/sdb
```


### 6. Format the specified partition with the `ext4` file system.
```bash
# Place for future command
```


### 7. Display the file system label of the specified partition.
```bash
# Place for future command
```


### 8. Change the label of an `ext4` file system.

```bash
# Place for future command
```

### 9. Display all currently active mount points.

```bash
# Place for future command
```


### 10. Mount a file system using its UUID.

```bash
# Place for future command
```

### 11. Mount the NFS share available at `192.168.1.100:/share`.

```bash
# Place for future command
```
