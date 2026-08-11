# Text processing (and log analysis)


## Used commands
- `grep`
- `egrep`
- `awk`
- `cut`
- `sort`
- `uniq`
- `wc`
- `sed`
- `diff`

# Tasks

## 1. Search for the text `ERROR` in all `.log` files under `/var/log`.
```bash
grep "ERROR" /var/log/*.log
```
or for recursive search
```bash
grep -ri --include="*.log" "error" /var/log
```

## 2. Display all lines that begin with the word `root`.
```bash
grep "^root" /etc/passwd
```

## 3. Display all lines that do **not** contain the word `ssh`.
```bash
grep -v "ssh" /etc/passwd
```

## 4. Count the number of occurrences of the word `ERROR` in error.log file.
```bash
grep -o "ERROR" errors.log | wc -l
```

## 5. Display only the username column from the `/etc/passwd` file.
```bash
cut -d: -f1 /etc/passwd
```
> **Note:** **-d** states the delimiter and **-f** states the field number
## 6. Display only unique usernames from a text file.
```bash
cut -d: -f1 /etc/passwd | sort | uniq 
```

## 7. Sort a file by the second column.
```bash
sort -t: -k2 /etc/passwd
```

## 8. Replace every occurrence of `root` with `toor` in a results.txt file. Create backup.
```bash
sed -i.bak 's/root/toor/g' ~/Tasks/results.txt
```

## 9. Display the ten most frequently occurring words in a results.txt file.
```bash
cat results.txt | tr -cs '[:alnum:]' '\n' | sort | uniq -c | sort -nr | head -10
```
> **Note:**
-tr (translates characters), c (complements everything except alphanumeric characters), s (squeezes repeated delimiters), 
'[:alnum:]' (letters and digits [alphanumeric]), '\n' (replace delimiters with newlines).

## 10. Display only unique IP addresses from a log file.
```bash
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' file.log | sort -u
```

## 11. Compare two text files and display their differences.
```bash
diff file.log errors.log
```

## 12. Sort a text file and remove duplicate lines.
```bash
sort file.txt | uniq
```
