# Archiving and Compression
## Used commands

* `tar`
* `xz`
* `7z`

# Tasks

## 1. Create a compressed '.xz' archive containing the `/etc` directory.

```bash
sudo tar -cJf backup.tar.xz /etc
```
Now it's possible to extract it using:
```bash
sudo tar -xJf backup.tar.xz
```

## 2. Display the contents of the `backup.tar.xz` archive without extracting it.

```bash
tar -tJf backup.tar.xz
```

## 3. Create an archive containing only files modified within the last **7 days**.

```bash
sudo find /etc -type f -mtime -7 -print0 | sudo tar --null -T - -cJf recent_files.tar.xz
```
>>**Note**: "--null -T -" tells tar that the filenames it receives are separated by a NULL character (\0) instead of a newline.
## 4. Find all files with the `.log` extension and compress them using `7zip`.
```bash
find . -type f -name "*.log" -exec 7z a my_logs.7z {} +
```
Now it's possible to extract them using:
```bash
7z x my_logs.7z
```

## 5. Compress report.txt file.
```bash
xz -k report.txt
```
File can be decompressed using:
```bash
unxz -d report.txt.xz
```
