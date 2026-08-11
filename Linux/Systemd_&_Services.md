# Systemd and Services

## Used commands

* `systemctl`
* `journalctl`
* `systemd-analyze`

# Tasks

## 1. Start the `sshd` service without adding it to aitomatic start during system boot.

```bash
sudo systemctl start ssh
```

## 2. Configure the `sshd` service to start automatically at system boot.

```bash
sudo systemctl enable ssh
```

## 3. Check the `sshd` service status.

```bash
systemctl status ssh
```

## 4. Display all `systemd` units that failed during the last system boot.

```bash
systemctl --failed
```

## 5. Display the dependencies of the `sshd.service` unit.

```bash
systemctl list-dependencies ssh.service
```

## 6. Display the system total boot time.

```bash
systemd-analyze blame
```
