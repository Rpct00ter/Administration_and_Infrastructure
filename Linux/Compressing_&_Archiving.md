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

