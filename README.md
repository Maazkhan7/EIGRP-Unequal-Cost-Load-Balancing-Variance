<div align="center">

# ⚖️ EIGRP Unequal-Cost Load Balancing & Variance

### Feasible Successor · Variance · Traffic Sharing · DUAL — Cisco EIGRP AS 1

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Protocol](https://img.shields.io/badge/Protocol-EIGRP-blue?style=for-the-badge)
![AS](https://img.shields.io/badge/AS-1-orange?style=for-the-badge)
![Variance](https://img.shields.io/badge/Variance-2-brightgreen?style=for-the-badge)
![UCMP](https://img.shields.io/badge/Load%20Balancing-Unequal--Cost-purple?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20%2F%20CCNP-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

</div>

---

## 📌 Project Overview

EIGRP is the **only mainstream IGP that can load-balance across unequal-cost paths**. This lab proves it step by step.

Two Cisco 2911 routers (**R1**, **R2**) are connected by **two parallel serial links**. The lab moves through three stages:

| Stage | Link 1 (Se0/0/0) | Link 2 (Se0/0/1) | Variance | Result |
|:-----:|:----------------:|:----------------:|:--------:|:-------|
| 1️⃣ **Equal cost** | 1000 Kbps | 1000 Kbps | 1 | **2 successors**, both links used |
| 2️⃣ **Unequal cost** | 1000 Kbps | **500 Kbps** | 1 | **1 successor**, 2nd path becomes a backup — **only Link 1 used** |
| 3️⃣ **Variance applied** | 1000 Kbps | 500 Kbps | **2** | **Both links used again** (unequal-cost load balancing) |

> 🎯 **Core idea:** A slower path can still carry traffic — if it is **loop-free (feasible)** and its metric is **within `variance × best metric`**.

---

## 🎯 Objectives

- ✅ Observe **equal-cost load balancing** (2 successors)
- ✅ Break the balance by changing `bandwidth` on one link
- ✅ See how the slower path is demoted to a **Feasible Successor**
- ✅ Calculate the **variance multiplier** from the two metrics
- ✅ Configure `variance 2` and watch **both paths return** to the routing table
- ✅ Prove the behavior with **traceroute** at each stage
- ✅ Understand the **feasibility condition** (RD < FD)
- ✅ Verify `show ip protocols` (variance, maximum paths, K-values, AD)

---

## 🗺️ Network Topology

```
   192.168.1.0/24                                                    192.168.2.0/24
                     Link 1 : 1.1.1.0/24 (Se0/0/0)  — 1000 Kbps
                   ┌───────────────────────────────────────────────┐
                   │ .1                                         .2 │
PC1 ─┐          ┌───┴──┐                                        ┌───┴──┐          ┌─ PC4
PC2 ─┼─ SW0 ────┤  R1  │                                        │  R2  ├──── SW1 ─┼─ PC5
PC3 ─┘   Gi0/0  │ 2911 │                                        │ 2911 │  Gi0/0   └─ PC6
         .1     └───┬──┘                                        └───┬──┘   .1
                   │ .1                                         .2 │
                   └───────────────────────────────────────────────┘
                     Link 2 : 2.1.1.0/24 (Se0/0/1)  — 500 Kbps
```

> 📷 Add your screenshot: `![Topology](images/topology.png)`

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Mask | Connected To | Bandwidth |
|:------:|:---------:|:----------:|:----:|:------------:|:---------:|
| **R1** | Gi0/0 | 192.168.1.1 | 255.255.255.0 | Switch0 (LAN 1) | — |
| **R1** | Se0/0/0 | 1.1.1.1 | 255.255.255.0 | R2 Se0/0/0 | **1000 Kbps** |
| **R1** | Se0/0/1 | 2.1.1.1 | 255.255.255.0 | R2 Se0/0/1 | **1000 → 500 Kbps** |
| **R2** | Gi0/0 | 192.168.2.1 | 255.255.255.0 | Switch1 (LAN 2) | — |
| **R2** | Se0/0/0 | 1.1.1.2 | 255.255.255.0 | R1 Se0/0/0 | **1000 Kbps** |
| **R2** | Se0/0/1 | 2.1.1.2 | 255.255.255.0 | R1 Se0/0/1 | **1000 → 500 Kbps** |
| PC1 / PC2 / PC3 | NIC | 192.168.1.2 / .3 / .4 | 255.255.255.0 | Switch0 | — |
| PC4 / PC5 / PC6 | NIC | 192.168.2.2 / .3 / .4 | 255.255.255.0 | Switch1 | — |

---

## ⚙️ Configuration

### 🔹 Base EIGRP (both routers)

```bash
! R1                                      ! R2
router eigrp 1                            router eigrp 1
 network 1.1.1.0 0.0.0.255                 network 1.1.1.0 0.0.0.255
 network 2.1.1.0 0.0.0.255                 network 2.1.1.0 0.0.0.255
 network 192.168.1.0 0.0.0.255             network 192.168.2.0 0.0.0.255
 no auto-summary                           no auto-summary
```

### 1️⃣ Stage 1 — Equal bandwidth on both links (R1 and R2)

```bash
interface Serial0/0/0
 bandwidth 1000
interface Serial0/0/1
 bandwidth 1000
```

### 2️⃣ Stage 2 — Lower the bandwidth of Link 2 (R1 **and** R2)

```bash
interface Serial0/0/1
 bandwidth 500
```

> ⚠️ `bandwidth` is a **metric-only** command — it does **not** change the real link speed. It is the standard way to influence EIGRP path selection.

### 3️⃣ Stage 3 — Enable unequal-cost load balancing

```bash
router eigrp 1
 variance 2
```

> Apply the same on **R2** if you want the return traffic (LAN 2 → LAN 1) to use both links too.

---

## ✅ Verification — Stage by Stage

### 🟢 Stage 1: Equal Cost → 2 Successors

**Interfaces** — `show interfaces serial 0/0/0` and `0/0/1` on R1

```
Serial0/0/0 ... Internet address is 1.1.1.1/24
  MTU 1500 bytes, BW 1000 Kbit, DLY 20000 usec
Serial0/0/1 ... Internet address is 2.1.1.1/24
  MTU 1500 bytes, BW 1000 Kbit, DLY 20000 usec          ◄── Same bandwidth & delay
```

**Neighbors** — `show ip eigrp neighbors` (R1)

| H | Address | Interface | Hold | Uptime | SRTT | RTO | Q | Seq |
|:-:|:-------:|:---------:|:----:|:------:|:----:|:---:|:-:|:---:|
| 0 | 1.1.1.2 | Se0/0/0 | 11 | 00:01:54 | 40 | 1000 | 0 | 22 |
| 1 | 2.1.1.2 | Se0/0/1 | 14 | 00:01:32 | 40 | 1000 | 0 | 24 |

**Topology table** — `show ip eigrp topology` (R1)

```
P 1.1.1.0/24, 1 successors, FD is 3072000
        via Connected, Serial0/0/0
P 2.1.1.0/24, 1 successors, FD is 3072000
        via Connected, Serial0/0/1
P 192.168.1.0/24, 1 successors, FD is 28160
        via Connected, GigabitEthernet0/0
P 192.168.2.0/24, 2 successors, FD is 3074560        ◄── 🔥 2 SUCCESSORS
        via 1.1.1.2 (3074560/28160), Serial0/0/0
        via 2.1.1.2 (3074560/28160), Serial0/0/1
```

**Traceroute** — `traceroute 192.168.2.1` run repeatedly from R1

```
1   1.1.1.2   ...        ◄── Link 1
1   2.1.1.2   ...        ◄── Link 2
1   1.1.1.2   ...        ◄── Link 1
1   2.1.1.2   ...        ◄── Link 2
```

✅ Traffic **alternates** between both links → **equal-cost load balancing** confirmed.

---

### 🟠 Stage 2: Bandwidth 500 on Link 2 → Only One Successor

**Topology table** (R1)

```
P 1.1.1.0/24, 1 successors, FD is 3072000
        via Connected, Serial0/0/0
P 2.1.1.0/24, 1 successors, FD is 5632000            ◄── Link 2 is now more expensive
        via Connected, Serial0/0/1
P 192.168.2.0/24, 1 successors, FD is 3074560        ◄── 🔥 JUST ONE SUCCESSOR
        via 1.1.1.2 (3074560/28160), Serial0/0/0     ◄── Successor
        via 2.1.1.2 (5634560/28160), Serial0/0/1     ◄── Feasible Successor (backup)
```

**Routing table** (R1)

```
D    192.168.2.0/24 [90/3074560] via 1.1.1.2, 00:09:20, Serial0/0/0     ◄── only ONE path
```

**Traceroute** — every run now goes through **1.1.1.2** only.

🧠 **Why is the second path still in the topology table?**
Its **RD (28160) < FD of the successor (3074560)** → it passes the **feasibility condition**, so it is kept as a **Feasible Successor** (loop-free backup) — but it's **not installed** because its metric is higher than the best path and `variance` is still 1.

---

### 🔴 Stage 3: `variance 2` → Unequal-Cost Load Balancing

**Check current variance** — `show ip protocols` (R1) **before the change**

```
Routing Protocol is "eigrp 1"
  Metric weight K1=1, K2=0, K3=1, K4=0, K5=0
  Distance: internal 90 external 170
  Maximum path: 4
  Maximum metric variance 1                          ◄── default
  Routing for Networks: 1.0.0.0 / 192.168.1.0 / 2.0.0.0
  Routing Information Sources:
    Gateway   Distance
    1.1.1.2   90
    2.1.1.2   90
```

**Calculate the multiplier**

```
Metric of backup path / Metric of best path
   5,634,560 / 3,074,560 = 1.83   →  round UP  →  variance 2
```

**Apply it**

```bash
R1(config)# router eigrp 1
R1(config-router)# variance 2
```
```
%DUAL-5-NBRCHANGE: IP-EIGRP 1: Neighbor 2.1.1.2 (Serial0/0/1) is up: new adjacency
%DUAL-5-NBRCHANGE: IP-EIGRP 1: Neighbor 1.1.1.2 (Serial0/0/0) is up: new adjacency
```

**Topology table** (R1)

```
P 192.168.2.0/24, 2 successors, FD is 3074560        ◄── 🔥 both paths active
        via 1.1.1.2 (3074560/28160), Serial0/0/0
        via 2.1.1.2 (5634560/28160), Serial0/0/1
```

**Routing table** (R2) — **different metrics, both installed**

```
D    192.168.1.0/24 [90/3074560] via 1.1.1.1, 00:00:09, Serial0/0/0
                    [90/5634560] via 2.1.1.1, 00:00:08, Serial0/0/1
```

**Neighbors** (R2)

| H | Address | Interface | Hold | Uptime | SRTT | RTO | Q | Seq |
|:-:|:-------:|:---------:|:----:|:------:|:----:|:---:|:-:|:---:|
| 0 | 1.1.1.1 | Se0/0/0 | 11 | 00:00:17 | 40 | 1000 | 0 | 39 |
| 1 | 2.1.1.1 | Se0/0/1 | 12 | 00:00:16 | 40 | 1000 | 0 | 40 |

**Traceroute** (R2 → 192.168.1.1)

```
Path A :  1   1.1.1.1   ...      ◄── via Link 1
Path B :  1   2.1.1.1   ...      ◄── via Link 2
```

✅ Traffic now uses **both links** even though their metrics are **different**.

> 📝 *Note:* Packet Tracer reports this state as "2 successors". On real IOS hardware the topology table typically keeps showing 1 successor + 1 feasible successor, while the **routing table** shows both routes once variance allows it.

---

## 🧮 Metric Calculations (matches the router output)

```
EIGRP metric = 256 × ( 10⁷ / lowest BW (Kbps)  +  total delay (tens of µs) )
```

| Path | Lowest BW | BW term | Delay term | Total × 256 | Router shows |
|:-----|:---------:|:-------:|:----------:|:-----------:|:------------:|
| Link 1 (1000 Kbps) – connected | 1000 | 10,000 | 2000 | **3,072,000** | ✅ 3072000 |
| Link 1 → remote LAN | 1000 | 10,000 | 2000 + 10 | **3,074,560** | ✅ 3074560 |
| Link 2 (500 Kbps) – connected | 500 | 20,000 | 2000 | **5,632,000** | ✅ 5632000 |
| Link 2 → remote LAN | 500 | 20,000 | 2000 + 10 | **5,634,560** | ✅ 5634560 |

**Variance check**

```
Variance × best FD  =  2 × 3,074,560  =  6,149,120
Backup path metric  =                      5,634,560   ≤ 6,149,120  ✅  → installed
(with variance 1:  3,074,560  <  5,634,560  ❌  → not installed)
```

---

## 📐 The Two Rules for Unequal-Cost Load Balancing

A path is installed with `variance` **only if both** are true:

| # | Rule | In this lab |
|:-:|:-----|:------------|
| 1️⃣ | **Feasibility condition:** RD of the path **<** FD of the successor | 28160 < 3074560 ✅ |
| 2️⃣ | **Variance condition:** metric of the path **≤** variance × FD of the successor | 5634560 ≤ 6149120 ✅ |

```
Successor            → best path, always installed
Feasible Successor   → loop-free backup (RD < FD)
Variance             → lets a Feasible Successor ALSO carry traffic
```

---

## 🧠 Core Concepts

| Term | Meaning |
|:-----|:--------|
| **Successor** | Next hop with the lowest metric (installed in routing table) |
| **Feasible Distance (FD)** | Best total metric from this router to the destination |
| **Reported Distance (RD)** | Metric the neighbor advertises for the destination |
| **Feasible Successor** | Backup next hop with **RD < FD** |
| **Variance** | Multiplier that decides how much worse a feasible path may be and still be used |
| **maximum-paths** | Max number of routes installed per prefix (default **4**) |
| **Traffic sharing** | Load is split **inversely proportional to the metric** (better path carries more) |

---

## 🔄 Equal vs Unequal Load Balancing

| Feature | Equal-Cost (ECMP) | Unequal-Cost (Variance) |
|:--------|:-----------------:|:-----------------------:|
| Metrics | Identical | Different |
| Configuration needed | None (default) | `variance N` |
| Condition | Equal FD | Feasible + within variance × FD |
| Traffic split | Even | **Proportional to metric** |
| Link utilisation | Good if links equal | **Better when links differ** |
| Supported by OSPF/RIP | ✅ | ❌ **EIGRP only** |

---

## 🎓 Key Takeaways

1. 🔸 Equal metrics → **multiple successors**, traffic alternates across both links
2. 🔸 Lowering `bandwidth` on one link raises its metric and **removes it from the routing table**
3. 🔸 The slower path stays in the topology table as a **Feasible Successor** (RD < FD)
4. 🔸 Variance multiplier = **⌈ backup metric ÷ best metric ⌉** → 5634560 ÷ 3074560 = 1.83 → **2**
5. 🔸 `variance` is **a multiplier**, not a metric value; default is **1**
6. 🔸 Only **feasible** paths can be used — variance can't create loops
7. 🔸 `bandwidth` changes the **metric only**, not the physical speed
8. 🔸 Changing bandwidth/variance may reset the adjacency in Packet Tracer (`%DUAL-5-NBRCHANGE`)
9. 🔸 Traceroute is a quick way to **prove** which paths forwarding really uses

---

## 🛠️ Useful Commands

```bash
show ip eigrp neighbors
show ip eigrp topology
show ip eigrp topology all-links      # include non-feasible paths
show ip route eigrp
show ip route 192.168.2.0             # shows traffic share count per path (when supported)
show ip protocols                     # variance, maximum-paths, K-values, AD
show interfaces serial 0/0/1          # BW and DLY used in the metric
traceroute 192.168.2.1
```

---

## 🧪 Challenges

1. Set `variance 3` and compare behavior
2. Change Link 2 to **100 Kbps** — what multiplier would you need, and does it still pass the feasibility condition?
3. Add `maximum-paths 1` and watch the second path disappear
4. Shut down Link 1 — how fast does traffic fail over to the feasible successor?

---

## 📁 Repository Structure

```
📦 EIGRP-Unequal-Load-Balancing-Variance
 ┣ 📜 README.md
 ┣ 📂 images
 ┃ ┣ 🖼️ topology.png
 ┃ ┣ 🖼️ 01-bandwidth-config.png
 ┃ ┣ 🖼️ 02-equal-cost-2-successors.png
 ┃ ┣ 🖼️ 03-interface-bandwidth.png
 ┃ ┣ 🖼️ 04-traceroute-equal.png
 ┃ ┣ 🖼️ 05-bandwidth-500.png
 ┃ ┣ 🖼️ 06-one-successor.png
 ┃ ┣ 🖼️ 07-traceroute-one-path.png
 ┃ ┣ 🖼️ 08-show-ip-protocols.png
 ┃ ┣ 🖼️ 09-variance-2.png
 ┃ ┗ 🖼️ 10-r2-unequal-paths.png
 ┗ 📂 lab-file
   ┗ 📦 EIGRP-Unequal-Load-Balancing.pkt
```

---

## 🔗 Series Navigation

⬅️ **Previous:** EIGRP Successor – 2 Paths (Equal-Cost Load Balancing)
⏮️ **Start:** EIGRP Successor – Single Path

`EIGRP` · `DUAL` · `Variance` · `Unequal-Cost Load Balancing` · `Feasible Successor` · `Feasibility Condition` · `Metric Calculation` · `Traffic Engineering`

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-Certified Network & NOC Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/maazkhanms)

---

<div align="center">

⭐ **If this lab helped you, star the repository!** ⭐

*Part of my ongoing Cisco Packet Tracer lab series — routing, switching, security & more.*

</div>
