# Files and Directories Management

## Used commands
- `ls`
- `cp`
- `mv`
- `rm`
- `mkdir`
- `touch`
- `ln`
- `find`

# Tasks

## 1. Create the `/opt/projects` directory, including any missing parent directories.
```bash
sudo mkdir -p /opt/projects
```

## 2. Create an empty file named `report.txt` in the `/opt/projects` directory.
```bash
sudo touch report.txt
```

## 3. Copy the `report.txt` file from `/opt/projects` to the `/tmp` directory.
```bash
cp /opt/projects/report.txt /tmp/
```

## 4. Create a new directory named `Tasks` in your home directory, then copy `report.txt` into it and rename it to `raport.txt`.
```bash
mkdir Tasks

cp /opt/projects/report.txt ~/Tasks/raport.txt
```

## 5. Move the `raport.txt` file from `~/Tasks` to `/tmp`.
```bash
mv ~/Tasks/raport.txt /tmp/
```

## 6. Find all files with the `.log` extension in the `/var` directory.
```bash
sudo find /var -type f -name "*.log"
```

## 7. Find all files larger than **500 MB**.
```bash
sudo find / -type f -size +500M
```

## 8. Find all files modified within the last **24 hours**.
```bash
find / -type f -mtime -1
```
(or in minutes):
```bash
find / -type f -mmin -1440
```

## 9. Find all files owned by the `john` user.
```bash
find / -type f -user john
```

## 10. Find all empty files in the `/home` directory.
```bash
find /home -type f -empty
```

## 11. Find all symbolic links in the `/etc` directory.
```bash
find /etc -type l
```

## 12. Create a symbolic link to the `/var/log/messages` file.
```bash
ln -s /var/log/messages ~/messages
```

## 13. Remove a symbolic link to the `/var/log/messages` file.
```bash
rm ~/messages
```

## 14. Find all files larger than **1 GB** and display their sizes.
```bash
sudo find / -type f -size +1G -exec ls -lh {} \;
```
> **Note:** **{}** is replaced with the current file and **\;** marks the end of the command.
## 14. Remove all files with the `*.tmp` extension in the ~\Downloads directory.
```bash
find ~\Downloads -type f -name "*.tmp" -delete
```
## 15. Remove the `/opt/projects` directory along with all of its contents.
```bash
sudo rm -r /opt/projects
```
> **Note:** **-r** means recursive and can be used with **-f** which means force (-rf)
