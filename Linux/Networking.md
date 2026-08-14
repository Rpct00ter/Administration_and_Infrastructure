# Networking
## Used commands

* `ip`
* `ss`
* `ping`
* `dig`
* `nmap`
* `arp`
* `tcpdump`
* `traceroute`
* `traceroute`

# Tasks

## 1. Display the IP addresses of all network interfaces.

```bash
ip addr
```

## 2. Display all TCP ports currently in use

```bash
ss -tn
```

## 3. Display the system's routing table.

```bash
ip route
```

## 4. Test connectivity to a remote host and determine whether packets are being lost.

```bash
ping -c 10 8.8.8.8
```

## 5. Find the process listening on port `8080`.

```bash
sudo ss -lntp | grep ':8080'
```

## 6. Display all network connections established by the `sshd` process.

```bash
sudo ss -p | grep sshd
```

## 7. Add a static route to the `10.10.20.0/24` network.

```bash
sudo ip route add 10.10.20.0/24 via <GATEWAY> dev <INTERFACE>
```

## 8. Display the statistics of the specified network interface.

```bash
ip -s link show <INTERFACE>
```

## 9. Test DNS name resolution for the `example.com` domain.

```bash
dig example.com
```

## 10. Display the ARP table cache.

```bash
ip neigh
```
or older
```bash
arp -a
```
## 11. Capture 20 packets on the `eth0` interface using `tcpdump`.

```bash
sudo tcpdump -i eth0 -c 20
```

## 12. Determine the route packets take to a remote host.

```bash
traceroute 8.8.8.8
```
## 13. Display all active network connections, including the local and remote addresses and ports.

```bash
ss -tun
```
## 14. Identify the default gateway used by the system.

```bash
ip route
```
## 15. Determine which services are listening locally and which ports are externally reachable (compare the results of ss with an Nmap scan of your own machine).

```bash
ss -lnt
```
compare it with
```bash
sudo nmap -O -sV <TARGET_IP>
```
## 16. Discover which hosts are currently online in your subnet.

```bash
sudo nmap -sn <SUBNET>/24
```
## 17. Perform a low-noise reconnaissance to determine which ports are open on another host in your network. Determine it's OS.

```bash
sudo nmap -sS --top-ports 100 <TARGET_IP>
```
```bash
sudo nmap -O -T1 <TARGET_IP>
```
