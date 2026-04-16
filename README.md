# SDN Mininet Simulation — Orange Problem

> **Course:** Computer Networks — UE24CS252B
> **Student Name:** Pavithra G C
> **USN:** PES2UG24CS346
> **Controller:** Ryu (OpenFlow v1.3)
> **Topology:** Single Switch, 3 Hosts

---

## Problem Statement

This project implements an **SDN-based network simulation** using **Mininet** and the **Ryu OpenFlow controller**. It demonstrates:

* Controller–switch interaction using the OpenFlow protocol
* Explicit flow rule design using **match–action logic**
* Dynamic packet forwarding (Learning Switch)
* Traffic filtering using a **Firewall/Access Control policy**
* Network behavior observation using ping and iperf

---

## SDN Behaviors Demonstrated

| Behavior                  | Description                                                          |
| ------------------------- | -------------------------------------------------------------------- |
| Learning Switch           | Controller learns MAC-to-port mappings and installs forwarding rules |
| Firewall / Access Control | Host h3 is blocked from communicating with any other host            |

---

## Network Topology

```
    h1 (10.0.0.1)
         |
    h2 (10.0.0.2) --- [s1 Switch] --- [Ryu Controller]
         |
    h3 (10.0.0.3)  ← BLOCKED
```

### 🔹 Topology Explanation

* The network consists of **3 hosts (h1, h2, h3)** and **1 switch (s1)**
* All hosts are connected to a **single OpenFlow switch (Open vSwitch)**
* The switch is controlled by the **Ryu SDN controller**
* Hosts **h1 and h2 are allowed to communicate**, while **h3 is blocked**

---

## Controller Logic

* The controller listens for **packet_in events** from the switch
* When a packet arrives:

  1. The controller checks the **source host or input port**
  2. It decides whether to **allow or block the traffic**

### Logic implemented:

* If packet is from **h3 (blocked host)**
  → Install **drop rule**

* If packet is from **h1 or h2 (allowed hosts)**
  → Install **forwarding rule**

 This enables **centralized control of network behavior**

---

## Match–Action Mechanism

In SDN, flow rules are based on **match–action principle**:

* **Match:** Identify packets based on fields like:

  * Source MAC address
  * Input port

* **Action:** Define what to do:

  * **Forward** → send packet to correct port
  * **Drop** → discard packet

### In this project:

* Match = packet from **h3**
  Action = **DROP**

* Match = packet from **h1/h2**
  Action = **FORWARD**

---

## Flow Rules

Flow rules are installed dynamically by the controller into the switch.

Each rule includes:

* Priority
* Match fields
* Action

### Implemented Flow Rules:

1. **Drop Rule (High Priority)**

   * Match: h3 traffic
   * Action: DROP

2. **Forwarding Rule**

   * Match: h1 ↔ h2 traffic
   * Action: OUTPUT to appropriate port

3. **Default Rule**

   * Action: Send packets to controller

### View flow rules:

```bash
sudo ovs-ofctl dump-flows s1
```

---

## Setup & Installation

### Prerequisites

* Ubuntu 20.04 / 22.04 (or WSL)
* Python 3.9
* Sudo privileges

### Step 1 — Update System

```bash
sudo apt update && sudo apt upgrade -y
```

### Step 2 — Install Mininet

```bash
sudo apt install mininet -y
```

### Step 3 — Install Python 3.9

```bash
sudo apt install python3.9 python3.9-venv -y
```

### Step 4 — Install Ryu Controller

```bash
python3.9 -m venv ryu-env
source ryu-env/bin/activate
pip install setuptools==58.0.0
pip install ryu
pip install eventlet==0.30.2
```

### Step 5 — Verify Installation

```bash
ryu-manager --version
```

---

## How to Run

### Terminal 1 — Start Controller

```bash
source ~/ryu-env/bin/activate
ryu-manager controller.py
```

### Terminal 2 — Start Mininet

```bash
sudo mn --controller=remote --topo single,3
```

---

## Test Scenarios

### Scenario 1 — Allowed Traffic

```bash
h1 ping h2 -c 4
```

Expected: 0% packet loss

---

### Scenario 2 — Blocked Traffic

```bash
h3 ping h1 -c 4
```

Expected: 100% packet loss

---

### Full Connectivity Test

```bash
pingall
```

Expected:

```
*** Results: 66% dropped (2/6 received)
```

---

### Throughput Test

```bash
h2 iperf -s &
h1 iperf -c h2
```

Expected: ~9.6 Gbits/sec

---

### Cleanup

```bash
sudo mn -c
```

---

## Performance Analysis

| Metric                | Value          |
| --------------------- | -------------- |
| Throughput (h1 → h2)  | ~9.6 Gbits/sec |
| Latency               | ~0.1 ms        |
| Packet Loss (h1 → h2) | 0%             |
| Packet Loss (h3 → h1) | 100%           |

---

## Proof of Execution

Screenshots included in `screenshots/` folder:

* Controller output
* pingall result
* Allowed ping (h1 → h2)
* Blocked ping (h3 → h1)
* iperf result
* Flow table

---

## Repository Structure

```
sdn-mininet-orange/
│
├── controller.py
├── topology.py
├── README.md
├── controller.png
├── pingall.png
├── h1_ping_h2.png
├── h3_ping_h1.png
├── iperf.png
└── flow_table.png
```

---

## Validation

* After restart, controller reapplies rules correctly
* h3 is always blocked
* h1 and h2 always communicate successfully

---

## Conclusion

This project demonstrates SDN using Mininet and Ryu controller.
The controller dynamically installs flow rules to allow or block traffic.
This shows how SDN enables programmable and centralized network control.

---

## References

1. https://mininet.org
2. https://ryu-sdn.org
3. https://github.com/mininet/mininet
