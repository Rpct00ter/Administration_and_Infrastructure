# Shell Environment
## Used commands

* `alias`
* `export`
* `env`
* `printenv`
* `grep`
* `/etc/profile`
* `~/.bashrc`
* `history`

# Tasks
## 1. Create an alias `ll` available only to the current user.

```bash
alias ll='ls -lah
```
>**Note**: Alias will disappear if the current shell section is terminated. To make it persistent it should be added '~/.bashrc'.
## 2. Add the `/opt/projects/scripts` directory to the `PATH` environment variable for the current session.

```bash
export PATH="$PATH:/opt/projects/scripts"
```

## 3. Check the current environment variables, especially the `EDITOR` variable.

```bash
env | grep EDITOR
```

## 4. Configure a persistent `EDITOR` environment variable set to `vim` for all users on the system.

```bash
sudo vim /etc/profile
```
```bash
#line added:
export EDITOR=vim
```

## 5. Display command history entries containing the word `docker`.

```bash
history | grep docker
```
