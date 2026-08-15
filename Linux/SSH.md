# SSH
## Used commands

* `ssh`
* `ssh-keygen`
* `scp`

## Tasks
### 1. Generate a new SSH key pair for the current user.

```bash
ssh-keygen -t ed25519 -C "Label"
```

### 2. Copy the specified file to a remote server using `scp`.

```bash
#syntax is: scp <LOCAL_FILE> <USER>@<REMOTE_IP>:<REMOTE_PATH>
scp /home/developer/test.txt developer@192.168.1.100:/home/developer/
```

### 3. Configure SSH key-based authentication for logging in to a remote server.

```bash
#Copies users public key to the remote server file with authorized keys.
#Syntax: ssh-copy-id <USER>@<REMOTE_IP>
ssh-copy-id developer@192.168.1.100
```

### 4. Connect to the remote host via ssh.

```bash
ssh <USER>@<REMOTE_IP>
```
>**Note**: If the ssh key-based authentication is configured, then  using password is not required. If key-based authentication is not configured typing password of the USER that we are trying to connect to will be necessary.
