# Redirection and Pipes

## Used commands
- `>`
- `>>`
- `2>`
- `2>&1`
- `tee`

# Tasks

## 1. Save the output of the `ls` command to a file.
```bash
ls -la > ~/Tasks/results.txt
```

## 2. Append the output of the `date` command to an existing file.
```bash
date >> ~/Tasks/results.txt
```

## 3. Save only error messages to a file named `errors.log`.
```bash
ls imaginary_file 2> errors.log
```

## 4. Redirect both standard output (`stdout`) and standard error (`stderr`) to the same output stream.
```bash
ls ~/ imaginary_file > output.log 2>&1
```
## 5. Hide generated error messages.
```bash
ls ~/ imaginary_file 2> /dev/null
```

## 5. Display the output on the screen and save it to a file at the same time using `tee`.
```bash
ping google.com | tee test_ping.log
```
