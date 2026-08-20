# Week 05 Portfolio – COIT20261 Network Technologies

**Name:** Karthik Murakonda
**Student ID:** 12317623
**Date:** 14 August 2026
**Unit:** COIT20261 – Network Technologies

---

## Task 1: Viewing Routing Tables

### Aim
Learn to view and interpret routing tables on Linux hosts and a Linux router. Test connectivity across two subnets connected through a router.

### Network Topology

```
[HOST-A] ──┐
           ├── Switch1 ── Linux-Router ── HOST-C
[HOST-B] ──┘
```

- Subnet 1: `10.1.1.0/24` — HOST-A, HOST-B and Linux-Router eth0
- Subnet 2: `10.1.2.0/24` — HOST-C and Linux-Router eth1

### IP Address Assignments

| Node | Interface | IP Address | Gateway | ip_forward |
|------|-----------|-----------|---------|------------|
| HOST-A | eth0 | 10.1.1.10/24 | 10.1.1.1 | 0 (host) |
| HOST-B | eth0 | 10.1.1.20/24 | 10.1.1.1 | 0 (host) |
| Linux-Router | eth0 | 10.1.1.1/24 | — | 1 (router) |
| Linux-Router | eth1 | 10.1.2.1/24 | — | 1 (router) |
| HOST-C | eth0 | 10.1.2.10/24 | 10.1.2.1 | 0 (host) |

### Activities Completed

1. Created project `View-Routes-12317623` with 5 nodes and 1 switch
2. Configured all IP addresses using `/etc/network/interfaces`
3. Enabled IP forwarding on Linux-Router:
```bash
sysctl net.ipv4.ip_forward=1
```

4. **Viewed routing tables** on each node using:
```bash
ip route show
```

5. **Linux-Router routing table** output (from screenshot):
```
10.1.1.0/24 dev eth0 scope link  src 10.1.1.1
10.1.2.0/24 dev eth1 scope link  src 10.1.2.1
```

6. **HOST-A routing table** output:
```
default via 10.1.1.1 dev eth0
10.1.1.0/24 dev eth0 scope link  src 10.1.1.10
```

7. **Pinged across subnets** from HOST-A to test routing:

| Destination | IP | TTL | Avg RTT | Result |
|------------|-----|-----|---------|--------|
| HOST-B (same subnet) | 10.1.1.20 | 64 | 0.203ms | ✅ 0% loss |
| HOST-C (other subnet) | 10.1.2.10 | 63 | 0.415ms | ✅ 0% loss |
| Router eth0 | 10.1.1.1 | 64 | 0.343ms | ✅ 0% loss |
| Router eth1 | 10.1.2.1 | 64 | 0.186ms | ✅ 0% loss |

### Key Observations from Routing Tables

- HOST-A has a **default route** via `10.1.1.1` (the router). Any traffic to unknown subnets is forwarded to the router
- Linux-Router has **no default route** — it only knows routes to subnets directly connected to it
- The **TTL value** of packets received at HOST-C from HOST-A is `63` (64 - 1), proving the packet traversed exactly 1 router

### Commands Used

```bash
# View routing table
ip route show

# Enable IP forwarding
sysctl net.ipv4.ip_forward=1

# Ping HOST-B (same subnet)
ping 10.1.1.20

# Ping HOST-C (cross-subnet)
ping 10.1.2.10

# Ping router interfaces
ping 10.1.1.1
ping 10.1.2.1
```

### Outputs

- **Project file:** `View-Routes-12317623.gns3project`
- **Network screenshot:** `View-Routes-12317623-network.png`
- **Ping screenshot:** `View-Routes-12317623-ping.png`
- **Router routing table:** `View-Routes-12317623-router-routing-table.png`
- **HOST-B routing table:** `View-Routes-12317623-host2-routing-table.png`
- **HOST-C routing table:** `View-Routes-12317623-host3-routing-table.png`

### Screenshots

![View Routes Network](View-Routes-12317623-network.png)
*Figure 1: Topology — HOST-A and HOST-B on Subnet 1 (10.1.1.0/24) via Switch1, connected to Linux-Router which links to HOST-C on Subnet 2 (10.1.2.0/24)*

![Ping Results](View-Routes-12317623-ping.png)
*Figure 2: HOST-A pinging HOST-B (10.1.1.20), HOST-C (10.1.2.10), and both router interfaces — all successful with 0% packet loss*

![Router Routing Table](View-Routes-12317623-router-routing-table.png)
*Figure 3: Linux-Router routing table showing two directly connected networks (10.1.1.0/24 on eth0 and 10.1.2.0/24 on eth1), with ip_forward=1 enabling packet forwarding*

### Learnings and Observations
- A router's job is to forward packets between subnets — it knows routes to each directly connected network
- Hosts only need a **default gateway** — they don't need to know all routes, just where to send unknown traffic
- The decrease in TTL from 64 to 63 on cross-subnet pings visually confirms that a router was traversed
- `ip route show` is the standard Linux command to view the kernel routing table
- Without `ip_forward=1`, even a node with two interfaces will not route packets between them

---

## Task 2: OSPF Dynamic Routing

### Aim
Configure and observe OSPF (Open Shortest Path First) dynamic routing using FRR (FRRouting) routers. Demonstrate automatic failover by simulating a link failure with NETem nodes.

### Network Topology

```
                    ┌── NETem1 ── FRR2 ──┐
Host1 ── FRR1 ──── ┤                    ├── FRR4 ── Host2
                    └── NETem2 ── FRR3 ──┘
         (eth0, eth1, eth2)         (eth0, eth1, eth2)
```

### IP Address Assignments

| Device | Interface | IP Address | Subnet |
|--------|-----------|-----------|--------|
| Host1 | eth0 | 10.0.0.1/24 | 10.0.0.0/24 |
| FRR1 | eth0 | 10.0.0.2/24 | 10.0.0.0/24 |
| FRR1 | eth1 | 10.0.1.1/24 | 10.0.1.0/24 |
| FRR1 | eth2 | 10.0.2.1/24 | 10.0.2.0/24 |
| FRR2 | eth0 | 10.0.1.2/24 | 10.0.1.0/24 |
| FRR2 | eth1 | 10.0.3.1/24 | 10.0.3.0/24 |
| FRR3 | eth0 | 10.0.2.2/24 | 10.0.2.0/24 |
| FRR3 | eth1 | 10.0.4.1/24 | 10.0.4.0/24 |
| FRR4 | eth0 | 10.0.3.2/24 | 10.0.3.0/24 |
| FRR4 | eth1 | 10.0.4.2/24 | 10.0.4.0/24 |
| FRR4 | eth2 | 10.0.5.1/24 | 10.0.5.0/24 |
| Host2 | eth0 | 10.0.5.2/24 | 10.0.5.0/24 |

### Activities Completed

#### Step 1 — Topology Setup
Built topology with 4 FRR 8.2.2 routers, 2 Alpine Linux hosts, and 2 NETem 0.4 link emulator nodes.

#### Step 2 — Configure IP Addresses
Set static IPs on all FRR routers using `/etc/network/interfaces` in each console, with `ip_forward=1`.

#### Step 3 — Configure OSPF on FRR1
```bash
vtysh
configure terminal
  router ospf
    network 10.0.0.0/24 area 0
    network 10.0.1.0/24 area 0
    network 10.0.2.0/24 area 0
    passive-interface eth0
  exit
exit
write memory
```

#### Step 4 — Configure OSPF on FRR2
```bash
vtysh
configure terminal
  router ospf
    network 10.0.1.0/24 area 0
    network 10.0.3.0/24 area 0
  exit
exit
write memory
```

#### Step 5 — Configure OSPF on FRR3
```bash
vtysh
configure terminal
  router ospf
    network 10.0.2.0/24 area 0
    network 10.0.4.0/24 area 0
  exit
exit
write memory
```

#### Step 6 — Configure OSPF on FRR4
```bash
vtysh
configure terminal
  router ospf
    network 10.0.3.0/24 area 0
    network 10.0.4.0/24 area 0
    network 10.0.5.0/24 area 0
    passive-interface eth2
  exit
exit
write memory
```

#### Step 7 — Verify OSPF Neighbours (FRR1)
```bash
vtysh -c "show ip ospf neighbor"
```

**Output:**
```
Neighbor ID  Pri  State      Up Time  Dead Time  Address    Interface
10.0.3.1     1    Full/DR    41.332s  38.639s    10.0.1.2   eth1:10.0.1.1
10.0.4.1     1    Full/Backup 10m21s  35.172s    10.0.2.2   eth2:10.0.2.1
```

Both FRR2 (Neighbor 10.0.3.1 via top path) and FRR3 (Neighbour 10.0.4.1 via bottom path) are **Full** state ✅

#### Step 8 — Verify OSPF Routes on FRR1
```bash
vtysh -c "show ip ospf route"
```

| Network | Cost | Next Hop | How |
|---------|------|---------|-----|
| 10.0.0.0/24 | 100 | eth0 | Directly connected |
| 10.0.1.0/24 | 100 | eth1 | Directly connected |
| 10.0.2.0/24 | 100 | eth2 | Directly connected |
| 10.0.3.0/24 | 200 | via 10.0.1.2, eth1 | Learned via OSPF |
| 10.0.4.0/24 | 200 | via 10.0.2.2, eth2 | Learned via OSPF |
| 10.0.5.0/24 | 300 | via 10.0.1.2 OR 10.0.2.2 | **Two equal paths!** |

#### Summary: OSPF Routes for ALL Routers

> This table summarises the routing table of each FRR router, showing which next node is used to reach each destination subnet.

| Router | Destination | Next Node / Interface |
|--------|------------|----------------------|
| **FRR1** | 10.0.0.0/24 | Directly connected (eth0) |
| **FRR1** | 10.0.1.0/24 | Directly connected (eth1) |
| **FRR1** | 10.0.2.0/24 | Directly connected (eth2) |
| **FRR1** | 10.0.3.0/24 | via FRR2 (10.0.1.2, eth1) |
| **FRR1** | 10.0.4.0/24 | via FRR3 (10.0.2.2, eth2) |
| **FRR1** | 10.0.5.0/24 | via FRR2 (10.0.1.2) OR FRR3 (10.0.2.2) |
| **FRR2** | 10.0.0.0/24 | via FRR1 (10.0.1.1, eth0) |
| **FRR2** | 10.0.1.0/24 | Directly connected (eth0) |
| **FRR2** | 10.0.2.0/24 | via FRR1 (10.0.1.1, eth0) |
| **FRR2** | 10.0.3.0/24 | Directly connected (eth1) |
| **FRR2** | 10.0.4.0/24 | via FRR4 (10.0.3.2, eth1) |
| **FRR2** | 10.0.5.0/24 | via FRR4 (10.0.3.2, eth1) |
| **FRR3** | 10.0.0.0/24 | via FRR1 (10.0.2.1, eth0) |
| **FRR3** | 10.0.1.0/24 | via FRR1 (10.0.2.1, eth0) |
| **FRR3** | 10.0.2.0/24 | Directly connected (eth0) |
| **FRR3** | 10.0.3.0/24 | via FRR4 (10.0.4.2, eth1) |
| **FRR3** | 10.0.4.0/24 | Directly connected (eth1) |
| **FRR3** | 10.0.5.0/24 | via FRR4 (10.0.4.2, eth1) |
| **FRR4** | 10.0.0.0/24 | via FRR2 (10.0.3.1, eth0) OR FRR3 (10.0.4.1, eth1) |
| **FRR4** | 10.0.1.0/24 | via FRR2 (10.0.3.1, eth0) |
| **FRR4** | 10.0.2.0/24 | via FRR3 (10.0.4.1, eth1) |
| **FRR4** | 10.0.3.0/24 | Directly connected (eth0) |
| **FRR4** | 10.0.4.0/24 | Directly connected (eth1) |
| **FRR4** | 10.0.5.0/24 | Directly connected (eth2) |

#### Step 9 — Test Connectivity (Host1 → Host2)
```bash
# From Host1
ping 10.0.5.2
```
**Result:** 3 packets transmitted, 3 received, 0% loss. TTL=61 (traversed 3 routers).

#### Step 10 — Traceroute Before Link Failure
```bash
traceroute 10.0.5.2
```
**Before failover (via top path through FRR2):**
```
1  10.0.0.2  (FRR1)   ~2ms
2  10.0.1.2  (FRR2)   ~4ms
3  10.0.3.2  (FRR4)   ~9ms
4  ^C (stopped)
```

#### Step 11 — Simulate Link Failure (Stop NETem1)
Right-clicked **NETem1** → **Stop**

Waited ~30–60 seconds for OSPF to detect the failure and reconverge.

#### Step 12 — Traceroute After Link Failure
```bash
traceroute 10.0.5.2
```
**After failover (via bottom path through FRR3):**
```
1  10.0.0.2  (FRR1)   ~0.9ms
2  10.0.2.2  (FRR3)   ~10ms
3  10.0.4.2  (FRR4)   ~4.7ms
4  10.0.5.2  (Host2)  ~3.8ms
```

OSPF automatically rerouted all traffic through FRR3 ✅

### Outputs

- **Project file:** `OSPF-Basics-12317623.gns3project`
- **Network screenshot:** `OSPF-Basics-12317623-network.png`
- **FRR1 neighbour table:** `OSPF-Basics-12317623-frr1-neighbors.png`
- **FRR2 routing table:** `OSPF-Basics-12317623-frr2-routing-table.png`
- **FRR3 routing table:** `OSPF-Basics-12317623-frr3-routing-table.png`
- **Traceroute before:** `OSPF-Basics-12317623-traceroute-before.png`
- **Traceroute after:** `OSPF-Basics-12317623-traceroute-after.png`

### Screenshots

![OSPF Network](OSPF-Basics-12317623-network.png)
*Figure 4: OSPF topology — Host1 and Host2 connected via 4 FRR routers with 2 redundant paths through NETem1 (top) and NETem2 (bottom)*

![FRR1 Neighbours](OSPF-Basics-12317623-frr1-neighbors.png)
*Figure 5: FRR1 OSPF neighbour table showing FRR2 (via eth1) and FRR3 (via eth2) both in Full state, confirming OSPF convergence. Also shows OSPF routing table with both paths to 10.0.5.0/24*

![FRR2 Routing Table](OSPF-Basics-12317623-frr2-routing-table.png)
*Figure 6: FRR2 routing table — shows O>* routes to all other subnets learned via OSPF*

![FRR3 Routing Table](OSPF-Basics-12317623-frr3-routing-table.png)
*Figure 7: FRR3 routing table — shows O>* routes to all other subnets learned via OSPF*

![Traceroute Before](OSPF-Basics-12317623-traceroute-before.png)
*Figure 8: Ping 10.0.5.2 succeeds (TTL=61), traceroute confirms path via FRR1 → FRR2(10.0.1.2) → FRR4(10.0.3.2) — top path through NETem1*

![Traceroute After](OSPF-Basics-12317623-traceroute-after.png)
*Figure 9: After NETem1 stopped, OSPF reconverged and traceroute now shows path via FRR1 → FRR3(10.0.2.2) → FRR4(10.0.4.2) → Host2(10.0.5.2) — bottom path through NETem2*

### Static vs Dynamic Routing Comparison

| Feature | Static Routes (Week 4) | OSPF Dynamic (Week 5) |
|---------|------------------------|----------------------|
| Configuration | Manual on each router | Configure once, auto-distributes |
| Failure handling | ❌ No automatic failover | ✅ Auto reroutes within 60s |
| Scalability | Hard (grows with nodes) | Easy (routers share info) |
| CPU overhead | None | Small |
| Route updates | Manual | Automatic |

### Learnings and Observations
- OSPF is a **link-state** routing protocol — each router knows the full network topology and calculates the best path
- OSPF routers discover each other automatically by sending **Hello packets** every 10 seconds
- After a link goes down, OSPF detects the failure via **Dead Time** (typically 40 seconds) and reconverges
- The `Full` state in the neighbour table means OSPF has fully synchronised its link-state database with that neighbour
- OSPF uses **cost** (related to link bandwidth) to determine the best path. Equal-cost paths can be used simultaneously (ECMP)
- FRR (FRRouting) is a modern open-source implementation of OSPF and other routing protocols used in real enterprise networks
- NETem nodes act as link emulators — stopping one simulates a cable cut, forcing OSPF to find an alternate path
- The difference in traceroute hop 2 (10.0.1.2 → 10.0.2.2) clearly shows OSPF choosing the bottom redundant path after the top path failed

