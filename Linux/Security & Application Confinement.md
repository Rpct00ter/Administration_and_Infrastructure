# Linux Security & Application Confinement
## Used commands

* `aa-status`
* `aa-enforce`
* `aa-complain`
* `firejail`
* `firejail --list`
* `getenforce`

## Tasks
### AppArmor
---
### 1. Check whether AppArmor is enabled and display its current status.

```bash
sudo aa-status
```

### 2. Switch an AppArmor profile for a selected application to Complain mode and then return it to Enforce mode.

```bash
sudo aa-complain <PROFILE>
sudo aa-enforce <PROFILE>
```

### Firejail
---
### 3. Run a selected application inside a Firejail sandbox.
```bash
firejail firefox
```


### 4. Display all applications currently running inside Firejail sandboxes.
```bash
firejail --list
```

### 5. Run an application inside Firejail with network access disabled.
```bash
firejail --net=none firefox
```


### SELinux
---
### 6. Check whether SELinux is installed and determine whether it is running in Enforcing, Permissive, or Disabled mode.
```bash
getenforce
```
