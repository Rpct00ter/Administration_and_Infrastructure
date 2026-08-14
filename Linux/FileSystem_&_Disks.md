# File Systems and Disks

## Used commands

* `df`
* `du`
* `lsblk`
* `blkid`
* `mount`
* `umount`
* `mkfs`
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


### 6. Format the specified partition with the `XFS` filesystem.
```bash
sudo mkfs.xfs /dev/sdb1
```

### 7. Display the filesystem label of the specified partition.
```bash
sudo blkid /dev/sdb1
```

### 8. Display all currently active mount points.

```bash
findmnt
#or
df -h
```

### 9. Mount a filesystem .

```bash
#First create a directory to access the system (we can name it whatever):
sudo mkdir -p /mnt/exampleName

#Then filesystem can be mounted:
sudo mount /dev/sdb1 /mnt/exampleName
```

### 10. Mount the NFS share available at `192.168.1.100:/share`.

```bash
#First local mount point should be created:
sudo mkdir -p /mnt/nfs_share

#Then nfs share can be mounted:
sudo mount -t nfs 192.168.1.100:/share /mnt/nfs_share
```
