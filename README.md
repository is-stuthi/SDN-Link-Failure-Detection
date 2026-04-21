# SDN Link Failure Detection & Performance Analysis

**COMPUTER NETWORKS ORANGE PROBLEM**
**Course:** B.Tech CSE 
**Student:** Mullunja Stuthi (PES1UG24CS283)

---

## 1. Problem Statement & Objective

The goal of this project is to implement an SDN-based solution using Mininet and the Ryu Controller to demonstrate real-time controller-switch interaction.

Specifically, this project focuses on:

* **Dynamic Forwarding:** Implementing a Learning Switch logic
* **Fault Tolerance:** Observing network behavior during a link failure and subsequent restoration
* **Performance Analysis:** Measuring network throughput and latency using iperf

---

## 2. Topology & Design Choice

* **Topology:** Linear, 3 Switches (`s1`, `s2`, `s3`) and 3 Hosts (`h1`, `h2`, `h3`)
  A linear topology is ideal for demonstrating link failure because it creates a clear *single point of failure* between switches.
  If the link between `s1` and `s2` goes down, `h1` is effectively isolated from the rest of the network, providing a clear **Normal vs. Failure** test scenario.

---

## 3. SDN Logic Implementation

The `controller.py` implements OpenFlow v1.3 logic:

* **Packet_In Handling:**
  The controller learns MAC addresses and maps them to switch ports

* **Flow Rule Installation:**
  Once a path is known, a priority 1 flow rule is installed in the switch's flow table to reduce controller overhead

* **Default Miss Rule:**
  A priority 0 rule is set during the handshake to send all unknown packets to the controller

---

## 4. Setup & Execution

### Start the Ryu Controller

```bash
ryu-manager controller.py
```

### Launch Mininet Topology

```bash
sudo mn --topo linear,3 --controller remote
```

### Verify Flow Rules

```bash
mininet> dpctl dump-flows
```

---

## 5. Performance Observation

| Metric       | Result                    | Tool            |
| ------------ | ------------------------- | --------------- |
| Connectivity | 0% dropped (Initial)      | `pingall`       |
| Throughput   | ~27.6 Gbits/sec           | `iperf h1 h3`   |
| Recovery     | 0% dropped (Post-Link Up) | `link s1 s2 up` |

### Analysis

The high throughput (~27.6 Gbits/sec) indicates that once the flow rules are installed in the switches (as seen in `dpctl dump-flows`), traffic is handled directly in the **data plane** without further controller intervention.

---

## 6. Test Scenarios

### Normal vs. Failure

* **Normal:**
  All hosts can communicate with each other (**0% packet loss**)

* **Failure:**

```bash
link s1 s2 down
```

Results in **66.7% packet loss**, as `h1` cannot reach `h2` or `h3`.

* **Recovery:**
```bash
link s1 s2 down
```
Results in **0% packet loss**.

---

### Flow Table Validation

Run:

```bash
dpctl dump-flows
```

