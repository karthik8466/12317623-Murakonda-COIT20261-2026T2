
# Week 02 Portfolio – COIT20261 Network Technologies

**Name:** Karthik Murakonda
**Student ID:** 12317623
**Date:** 13 August 2026
**Unit:** COIT20261 – Network Technologies

---

## Task 1: Setting Static IP Addresses

### Aim
Demonstrate three different approaches to configuring static IP addresses on Linux hosts in GNS3.

### Network Used
- **Project:** `Setting-IP-12317623`
- **Network:** `172.20.10.0/24`
- **Topology:** 4 Alpine Linux hosts connected to 1 Ethernet Switch (LAN)

### IP Address Assignments

| Host | Interface | IP Address | Method Used |
|------|-----------|-----------|-------------|
| AlpineLinux-1 | eth0 | 172.20.10.20/24 | GNS3 Configure menu (pre-start) |
| AlpineLinux-2 | eth0 | 172.20.10.21/24 | GNS3 Configure menu (pre-start) |
| AlpineLinux-3 | eth0 | 172.20.10.22/24 | Editing `/etc/network/interfaces` in console |
| AlpineLinux-4 | eth0 | 172.20.10.23/24 | `ip address add` command |

### Activities Completed

1. Created project `Setting-IP-12317623` with 4 Alpine Linux hosts and 1 Ethernet Switch
2. Connected all 4 hosts to the switch to form a LAN
3. Selected network address `172.20.10.0/24`

#### Method 1 – GNS3 Configure Menu (Hosts 1 & 2)
Right-clicked each node → Configure → Edit network configuration, and set:
```
auto eth0
iface eth0 inet static
        address 172.20.10.20
        netmask 255.255.255.0
        gateway 172.20.10.1
```
IP is applied automatically when the node starts.

#### Method 2 – Edit `/etc/network/interfaces` in Console (Host 3)
After starting the node, opened the console and used `nano` to edit the file, then reloaded networking:
```bash
nano /etc/network/interfaces
# Added:
auto eth0
iface eth0 inet static
    address 172.20.10.22
    netmask 255.255.255.0

# Then reloaded
ifdown eth0
ifup eth0
```

#### Method 3 – `ip address add` Command (Host 4)
```bash
ip address add 172.20.10.23/24 dev eth0
```
This applies immediately but is **not persistent** across reboots.

4. Verified all four hosts with `ip address show` on each console.

### Commands Used

```bash
# Method 3 - set IP immediately
ip address add 172.20.10.23/24 dev eth0

# Verify IP on all hosts
ip address show

# Method 2 - reload after editing interfaces file
ifdown eth0
ifup eth0
```

### Outputs

- **Project file:** `Setting-IP-12317623.gns3project`
- **Network screenshot:** `Setting-IP-12317623-network.png`
- **Host console screenshots:** `Setting-IP-12317623-host1.png` through `host4.png`

### Screenshots

![Network](Setting-IP-12317623-network.png)
*Figure 1: GNS3 network canvas showing 4 hosts connected to a switch*

![Host 1](Setting-IP-12317623-host1.png)
*Figure 2: AlpineLinux-1 showing IP 172.20.10.20/24 (Method 1 – GNS3 Configure menu)*

![Host 2](Setting-IP-12317623-host2.png)
*Figure 3: AlpineLinux-2 showing IP 172.20.10.21/24 (Method 1 – GNS3 Configure menu)*

![Host 3](Setting-IP-12317623-host3.png)
*Figure 4: AlpineLinux-3 showing IP 172.20.10.22/24 (Method 2 – interfaces file in console)*

![Host 4](Setting-IP-12317623-host4.png)
*Figure 5: AlpineLinux-4 showing IP 172.20.10.23/24 (Method 3 – ip address add)*

### Comparison of Three Methods

| Method | Persistent? | Requires Restart? | When Applied |
|--------|------------|-------------------|--------------|
| GNS3 Configure menu | ✅ Yes | Node start | Before start |
| Edit `/etc/network/interfaces` in console | ✅ Yes | `ifdown`/`ifup` | After start |
| `ip address add` | ❌ No | None (immediate) | After start |

### Learnings and Observations
- All three methods achieve the same result but differ in persistence and timing
- The GNS3 Configure menu actually edits the same `/etc/network/interfaces` file — the only difference is when it is applied
- The `ip address add` method is the fastest but settings are lost when the node restarts
- The `ifdown`/`ifup` commands are required to reload the interfaces file when changes are made after the node has already started

---

## Task 2: Testing Network Connectivity with Ping

### Aim
Learn the basics of ping to test if a device is reachable and to measure round-trip time (RTT).

### Activities Completed

#### Ping with No Options (Host A → Host B)
From AlpineLinux-1, pinged AlpineLinux-2 (`172.20.10.11`):
```bash
ping 172.20.10.11
```
Stopped after 6 responses were received (Ctrl-C).

**Results:**
- 6 packets transmitted, 6 received
- 0% packet loss
- RTT min/avg/max = 0.235/0.327/0.393 ms

#### Ping to Non-Existent IP
Pinged an IP address not in the network, observed 100% packet loss.

#### Ping with Options
Tested various options:
```bash
ping -c 5 172.20.10.11          # limit to 5 packets
ping -i 2 172.20.10.11          # 2-second interval between requests
ping -s 100 172.20.10.11        # 100-byte data payload
ping -c 3 -s 80 172.20.10.11    # combined: 3 packets, 80-byte payload
```

### Outputs

- `Ping-Basics-12317623-simple.png` – basic ping with no options
- `Ping-Basics-12317623-error.png` – ping to non-existent IP
- `Ping-Basics-12317623-options-count.png` – ping with `-c 5`
- `Ping-Basics-12317623-options-interval.png` – ping with `-i 2`
- `Ping-Basics-12317623-options-size.png` – ping with `-s 100`
- `Ping-Basics-12317623-options-combined.png` – combined options

### Screenshots

![Simple Ping](Ping-Basics-12317623-simple.png)
*Figure 6: Ping from AlpineLinux-1 to 172.20.10.11 – 6 packets, 0% loss, avg RTT 0.327ms*

### Understanding Ping Output

From the ping output:
```
6 packets transmitted, 6 received, 0% packet loss
round-trip min/avg/max = 0.235/0.327/0.393 ms
```
- **0% packet loss** confirms the destination is reachable
- **avg RTT of 0.327ms** is very low because both hosts are on the same LAN (no routing needed)
- The RTT measures the time for a request to reach the destination and a response to come back

### Learnings and Observations
- Ping uses ICMP (network layer), unlike Netcat which uses TCP/UDP (application layer)
- On Linux, ping runs indefinitely by default — Ctrl-C is needed to stop it
- The `-c` option is useful to automatically limit the number of packets
- When pinging a non-existent IP, 100% packet loss is reported as no response is received
- Very low RTT values (< 1ms) are expected for devices on the same physical/virtual LAN
