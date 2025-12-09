# Fat-Tree Data Center Network Simulation for Distributed Machine Learning

A comprehensive network simulation framework that emulates a Fat-Tree data center topology using Mininet to analyze the performance characteristics of distributed machine learning workloads, specifically CIFAR-10 gradient exchange patterns under various network conditions and congestion control protocols.

## Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Fat-Tree Topology](#fat-tree-topology)
- [Prerequisites and Dependencies](#prerequisites-and-dependencies)
- [Installation and Environment Setup](#installation-and-environment-setup)
- [Mininet Virtual Machine Configuration](#mininet-virtual-machine-configuration)
- [Network Controller Setup](#network-controller-setup)
- [Traffic Generation and Replay](#traffic-generation-and-replay)
- [Running Experiments](#running-experiments)
- [Measurement Methodology](#measurement-methodology)
- [Results and Analysis](#results-and-analysis)
- [Project Structure](#project-structure)
- [Technical Implementation Details](#technical-implementation-details)
- [Troubleshooting](#troubleshooting)
- [References](#references)

## Overview

This project implements and evaluates a three-tier Fat-Tree data center network topology to study the impact of network conditions on distributed machine learning traffic patterns. The simulation framework is built on Mininet, a network emulator that creates realistic virtual networks using actual kernel, switch, and application code.

### Research Objectives

1. Evaluate Fat-Tree topology performance for distributed ML gradient exchange workloads
2. Measure the impact of bandwidth constraints on training iteration latency
3. Compare TCP versus DCTCP (Data Center TCP) congestion control mechanisms
4. Analyze network throughput, latency, jitter, and bisection bandwidth characteristics
5. Validate the efficacy of Fat-Tree architecture for ML data center deployments

### Key Contributions

- Implementation of a k-ary Fat-Tree topology (k=4) with 20 switches and 16 hosts
- Traffic replay mechanism based on real CIFAR-10 distributed training traces
- Automated experiment framework for bandwidth and protocol comparison studies
- Comprehensive measurement and visualization pipeline
- Support for both simulated and actual distributed PyTorch training

## System Architecture

### Components

The system consists of several interconnected components:

1. **Mininet Network Emulator**: Creates virtual hosts, switches, and links in a Linux VM
2. **RYU OpenFlow Controller**: Manages switch forwarding tables using OpenFlow 1.3 protocol
3. **Fat-Tree Topology**: Three-tier Clos network with core, aggregation, and edge layers
4. **Traffic Replay System**: Client-server application that replays recorded gradient exchange patterns
5. **Measurement Infrastructure**: iperf, ping, and custom logging for performance metrics
6. **Visualization Tools**: Python scripts for generating plots and statistical summaries

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     RYU Controller                          │
│                  (OpenFlow 1.3 @ port 6653)                 │
└──────────────────────────┬──────────────────────────────────┘
                           │ Control Plane
                           │
┌──────────────────────────┴──────────────────────────────────┐
│                    Mininet Network                          │
│  ┌────────────────────────────────────────────────────┐    │
│  │            Core Layer (4 switches)                 │    │
│  │         c1          c2          c3          c4     │    │
│  └───┬────────┬────────┬────────┬────────┬────────┬───┘    │
│      │        │        │        │        │        │        │
│  ┌───┴────────┴────────┴────────┴────────┴────────┴───┐    │
│  │      Aggregation Layer (8 switches)               │    │
│  │   a5   a6   a7   a8   a9   a10  a11  a12         │    │
│  └───┬────┬────┬────┬────┬────┬────┬────┬───────────┘    │
│      │    │    │    │    │    │    │    │                │
│  ┌───┴────┴────┴────┴────┴────┴────┴────┴───────────┐    │
│  │           Edge Layer (8 switches)                 │    │
│  │   e13  e14  e15  e16  e17  e18  e19  e20         │    │
│  └───┬────┬────┬────┬────┬────┬────┬────┬───────────┘    │
│      │    │    │    │    │    │    │    │                │
│  ┌───┴────┴────┴────┴────┴────┴────┴────┴───────────┐    │
│  │           Hosts (16 total: h1 - h16)              │    │
│  │   Parameter Server (h16) ← → Workers (h1-h15)    │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

## Fat-Tree Topology

### Topology Theory

The Fat-Tree topology, proposed by Mohammad Al-Fares et al., is a Clos network architecture designed for data centers. It addresses the traditional three-tier architecture's oversubscription problem by providing full bisection bandwidth using commodity Ethernet switches.

### Mathematical Structure

For a k-ary Fat-Tree where k is the number of ports per switch:

- **Pods**: k pods
- **Core Switches**: (k/2)² switches
- **Aggregation Switches per Pod**: k/2 switches
- **Edge Switches per Pod**: k/2 switches
- **Hosts per Pod**: (k/2)² hosts
- **Total Hosts**: k³/4 hosts

### Implementation (k=4)

This project implements a k=4 Fat-Tree with the following characteristics:

```
Topology Parameters:
- Pods: 4
- Core Switches: 4 (c1, c2, c3, c4)
- Aggregation Switches: 8 total, 2 per pod (a5-a12)
- Edge Switches: 8 total, 2 per pod (e13-e20)
- Hosts: 16 total, 2 per edge switch (h1-h16)
```

### Switch Connectivity

**Core to Aggregation**: Each core switch connects to exactly one aggregation switch in each pod, ensuring multiple paths between pods.

**Aggregation to Edge**: Within each pod, every aggregation switch connects to every edge switch, providing multiple paths for intra-pod traffic.

**Edge to Hosts**: Each edge switch connects to exactly 2 hosts.

### Path Redundancy

The Fat-Tree provides multiple equal-cost paths between any pair of hosts:
- Intra-pod communication: Multiple paths through different aggregation switches
- Inter-pod communication: Multiple paths through different core switches
- Full bisection bandwidth: 1:1 oversubscription ratio

## Prerequisites and Dependencies

### Operating System

- Ubuntu 18.04 LTS or 20.04 LTS (recommended)
- Linux kernel 4.15 or later
- Minimum 2 GB RAM, 4 GB recommended
- 10 GB free disk space

### Required Software

**Core Components**:
- Python 3.6 or later
- Mininet 2.3.0 or later
- Open vSwitch (OVS) 2.9 or later
- OpenFlow 1.3 compatible switches

**Network Tools**:
- iperf (version 2.x)
- ping
- tc (traffic control from iproute2 package)
- tcpdump (optional, for packet capture)

**Python Packages**:
```
mininet
ryu (OpenFlow controller)
numpy
pandas
matplotlib
torch (PyTorch, for distributed training)
torchvision
csv
socket
struct
argparse
```

## Installation and Environment Setup

### Step 1: Install Mininet

```bash
# Clone Mininet repository
git clone https://github.com/mininet/mininet.git
cd mininet

# Install Mininet with all dependencies
sudo ./util/install.sh -a

# Verify installation
sudo mn --version
```

The `-a` flag installs:
- Mininet core
- OpenFlow reference implementation
- Open vSwitch
- Wireshark dissector
- POX controller (we will use RYU instead)

### Step 2: Install RYU Controller

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install -y python3-pip python3-dev

# Clone RYU repository
git clone https://github.com/faucetsdn/ryu.git
cd ryu

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install RYU
pip install -r tools/pip-requires
pip install .

# Verify installation
ryu-manager --version
```

### Step 3: Install Python Dependencies

```bash
# Install PyTorch (CPU version is sufficient for traffic generation)
pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu

# Install data analysis and visualization libraries
pip install numpy pandas matplotlib seaborn

# Install network tools via apt
sudo apt-get install -y iperf net-tools
```

### Step 4: Clone This Repository

```bash
# Clone the project
git clone https://github.com/andrew45110/Fat-Tree-Topology-.git
cd Fat-Tree-Topology-

# Set Python path
export PYTHONPATH=$HOME/mininet:$PYTHONPATH

# Make scripts executable
chmod +x run_bandwidth_tests.sh
chmod +x run_tcp_dctcp_compare.sh
chmod +x Fat-Tree-Data-Center-Topology/Code/*.py
```

## Mininet Virtual Machine Configuration

### Understanding Mininet's Operation

Mininet runs in a virtualized or containerized Linux environment. It creates network namespaces for each host, allowing multiple virtual hosts to coexist on a single physical machine while maintaining network isolation.

### Key Concepts

**Network Namespaces**: Each Mininet host runs in its own network namespace, providing isolated network stacks (interfaces, routing tables, firewall rules).

**Virtual Ethernet (veth) Pairs**: Links between hosts and switches are implemented as veth pairs, where one end is in the host namespace and the other is in the root namespace connected to the switch.

**Open vSwitch**: Switches are implemented using OVS, which provides OpenFlow protocol support for programmatic control.

**Process Execution**: Commands executed on Mininet hosts run as actual processes on the underlying Linux system, but within the appropriate network namespace.

### VM Networking Configuration

When running in a VM, ensure proper network configuration:

```bash
# Check available network interfaces
ip addr show

# Typical configuration for VirtualBox/VMware
# eth0: NAT (internet access)
# eth1: Host-only or bridged (management access)

# Verify routing
ip route show

# Test external connectivity
ping -c 3 8.8.8.8
```

### Resource Allocation

Recommended VM settings:
- CPUs: 2 cores minimum, 4 cores for better performance
- RAM: 2 GB minimum, 4 GB recommended
- Disk: 10 GB minimum
- Network Adapters: 2 (NAT + Host-only)

## Network Controller Setup

### RYU Controller Architecture

RYU is a component-based SDN framework written in Python. It provides:

1. **OpenFlow Protocol Support**: OpenFlow 1.0 through 1.5
2. **Event-Driven Architecture**: Handlers for packet-in, flow-removed, etc.
3. **Network Applications**: Pre-built apps for switching, routing, firewall
4. **RESTful API**: HTTP interface for network management

### Starting the RYU Controller

The controller must be running before starting Mininet:

```bash
# Activate RYU virtual environment
source ~/ryu/venv/bin/activate

# Start RYU with simple_switch_13 application
cd ~/ryu
ryu-manager ryu.app.simple_switch_13

# Output should show:
# loading app ryu.app.simple_switch_13
# loading app ryu.controller.ofp_handler
# instantiating app ryu.app.simple_switch_13 of SimpleSwitch13
# ...
```

### Controller Applications

**simple_switch_13**: Basic L2 learning switch using OpenFlow 1.3
- Learns MAC addresses from packet-in messages
- Installs flow rules for known destinations
- Floods packets to unknown destinations

**simple_switch_stp_13**: L2 switch with Spanning Tree Protocol
- Prevents loops in redundant topologies
- Required for Fat-Tree to avoid broadcast storms
- Use this for production experiments

```bash
# Start with STP support
ryu-manager ryu.app.simple_switch_stp_13
```

### Controller Configuration

RYU listens on:
- OpenFlow: TCP port 6653 (default OpenFlow port)
- WSGI server: TCP port 8080 (REST API)

Verify controller is listening:
```bash
sudo lsof -i :6653
# Should show ryu-manager listening
```

### Connecting Mininet to RYU

In the topology script, switches connect to RYU via RemoteController:

```python
from mininet.node import RemoteController

net = Mininet(
    topo=topo,
    controller=lambda name: RemoteController(
        name, ip='127.0.0.1', port=6653
    ),
    link=TCLink
)
```

## Traffic Generation and Replay

### CIFAR-10 Training Profile

The traffic patterns are derived from actual PyTorch distributed training of a ResNet model on CIFAR-10:

**Training Configuration**:
- Model: ResNet-18
- Dataset: CIFAR-10 (50,000 training images)
- Batch Size: 64 images per batch
- Epochs: Multiple epochs recorded
- Gradient Size: 248,024 bytes per iteration

**Captured Metrics**:
- Forward pass duration
- Backward pass duration
- Gradient tensor size
- Inter-iteration timing

### Traffic Profile Format

The `cifar_traffic_profile.csv` file contains:

```csv
interval_s,grad_bytes
0.033562,248024
0.030782,248024
0.027410,248024
...
```

- **interval_s**: Time to wait before next gradient exchange (simulates computation time)
- **grad_bytes**: Size of gradient data to transmit

### Traffic Replay Implementation

The `traffic_replay.py` script implements a simple client-server protocol:

**Server Mode** (Parameter Server - h16):
```python
# Listens for connections
# Receives gradient data
# Logs receive timestamps
```

**Client Mode** (Worker - h1):
```python
# Reads traffic profile CSV
# Waits for specified interval
# Sends gradient data (header + payload)
# Logs send timestamps
```

### Protocol Details

Message format:
```
[8-byte size header (big-endian uint64)][payload of <size> bytes]
```

This simulates:
1. Worker computes gradients (wait interval_s)
2. Worker sends gradients to parameter server
3. Parameter server receives and acknowledges
4. Repeat for all training iterations

### Running Traffic Replay Manually

```bash
# Start Mininet with Fat-Tree topology
sudo python3 Fat-Tree-Data-Center-Topology/Code/run_fat_tree.py

# In Mininet CLI:
mininet> h16 python3 traffic_replay.py --mode server --port 5000 &
mininet> h1 python3 traffic_replay.py --mode client --host 10.0.0.16 --port 5000 --csv cifar_traffic_profile.csv &
```

## Running Experiments

### Experiment 1: Bandwidth Impact Analysis

This experiment evaluates how bandwidth constraints affect gradient exchange latency.

**Script**: `run_bandwidth_tests.sh`

**Parameters**:
- Bandwidth levels: 5 Mbps, 10 Mbps, 20 Mbps
- Applied to: Core-to-aggregation links
- Queueing discipline: Token Bucket Filter (TBF)

**Execution**:
```bash
# Clean any previous Mininet state
sudo mn -c

# Ensure RYU controller is running in another terminal
# source ~/ryu/venv/bin/activate
# ryu-manager ryu.app.simple_switch_13

# Run bandwidth experiments
./run_bandwidth_tests.sh
```

**What the script does**:
1. Cleans Mininet environment
2. Creates result directory for each bandwidth setting
3. Launches Mininet with Fat-Tree topology
4. Applies bandwidth limits using tc (traffic control)
5. Starts iperf server on h16
6. Starts iperf client on h1 (measures throughput)
7. Runs traffic replay (h16 as server, h1 as client)
8. Collects latency data from logs
9. Parses and saves results to CSV
10. Repeats for next bandwidth setting

**Duration**: Approximately 10-15 minutes per bandwidth level

### Experiment 2: TCP vs DCTCP Comparison

This experiment compares standard TCP with Data Center TCP (DCTCP).

**Script**: `run_tcp_dctcp_compare.sh`

**Parameters**:
- Protocols: TCP (default), DCTCP (with ECN)
- Bandwidth: 10 Mbps (fixed)
- ECN marking threshold: Configured via tc

**DCTCP Configuration**:
```bash
# Enable ECN on hosts
sysctl -w net.ipv4.tcp_ecn=1

# Configure RED queueing with ECN marking
tc qdisc add dev eth0 root red \
    limit 1000000 \
    min 30000 \
    max 35000 \
    avpkt 1000 \
    burst 55 \
    ecn \
    adaptive \
    bandwidth 10mbit
```

**Execution**:
```bash
sudo mn -c
./run_tcp_dctcp_compare.sh
```

**Duration**: Approximately 20-30 minutes total

### Experiment 3: Custom Configuration

Run with custom parameters:

```bash
sudo PYTHONPATH=$HOME/mininet python3 Fat-Tree-Data-Center-Topology/Code/run_sim_fat_tree.py \
  --k 4 \
  --ps-host h16 \
  --worker-host h1 \
  --csv cifar_traffic_profile.csv \
  --port 5000 \
  --iperf-port 5001 \
  --iperf-duration 10 \
  --core-bw 15mbit \
  --qdisc tbf \
  --result-dir results/custom_15mbit \
  --debug
```

**Available Parameters**:

- `--k`: Fat-Tree parameter (number of ports per switch)
- `--ps-host`: Parameter server hostname
- `--worker-host`: Worker hostname
- `--csv`: Path to traffic profile CSV
- `--port`: Port for traffic replay
- `--iperf-port`: Port for iperf measurements
- `--iperf-duration`: Duration of iperf test in seconds
- `--core-bw`: Bandwidth limit for core links (e.g., 10mbit, 20mbit)
- `--qdisc`: Queue discipline (fifo, tbf, netem, dctcp)
- `--netem-args`: Network emulation parameters (e.g., "delay 10ms")
- `--ecn`: Enable Explicit Congestion Notification
- `--auto-exit`: Exit after experiment completes (no CLI)
- `--result-dir`: Directory to store results
- `--debug`: Enable verbose debugging output

### Automated Experiment Workflow

The `run_sim_fat_tree.py` script orchestrates the entire experiment:

1. **Initialize Topology**:
   - Create Fat-Tree topology
   - Connect to RYU controller
   - Start Mininet network

2. **Configure Network**:
   - Apply bandwidth limits to core links
   - Configure queue disciplines
   - Enable ECN if requested

3. **Verify Connectivity**:
   - Ping between all hosts
   - Wait for OpenFlow rules to populate

4. **Start Measurements**:
   - Launch iperf server on parameter server host
   - Launch traffic replay server on parameter server
   - Collect switch statistics

5. **Run Traffic Replay**:
   - Launch iperf client (background)
   - Launch traffic replay client
   - Monitor and log all activity

6. **Collect Results**:
   - Parse latency logs
   - Extract throughput from iperf
   - Collect switch port statistics
   - Save all data to result directory

7. **Cleanup**:
   - Stop Mininet
   - Clean up processes

## Measurement Methodology

### Latency Measurement

Latency is measured as the time between:
- Client sends gradient data
- Server acknowledges receipt

**Implementation**:
```python
# Client side
t_start = time.time()
sock.sendall(data)
# ... wait for ack ...
latency = time.time() - t_start
```

Latencies are logged per batch and saved to `latencies.csv`:
```csv
batch,latency_s
0,0.0234
1,0.0189
2,0.0245
...
```

### Throughput Measurement

Throughput is measured using iperf between worker (h1) and parameter server (h16):

```bash
# Server (h16)
iperf -s -p 5001

# Client (h1)
iperf -c <h16_ip> -p 5001 -t 10 -i 1
```

Output is parsed to extract:
- Average throughput (Mbps)
- Jitter (ms)
- Packet loss percentage

Saved to `throughput.csv`:
```csv
metric,value
throughput_mbps,9.87
jitter_ms,0.234
loss_percent,0.0
```

### Round-Trip Time (RTT)

RTT is measured using ping:

```bash
# From h1 to h16
ping -c 100 -i 0.2 <h16_ip>
```

Statistics extracted:
- Minimum RTT
- Average RTT
- Maximum RTT
- Standard deviation

### Switch Statistics

OpenFlow port statistics are collected before and after experiments:

```bash
# Query switch c1
ovs-ofctl -O OpenFlow13 dump-ports c1
```

Metrics:
- Packets transmitted/received
- Bytes transmitted/received
- Dropped packets
- Errors

### Measurement Challenges

**Clock Synchronization**: All measurements use the same physical clock (VM host) since Mininet hosts share the same kernel.

**Overhead**: Mininet introduces some overhead compared to physical hardware, but relative comparisons remain valid.

**Variability**: Multiple runs are recommended to account for system variability.

## Results and Analysis

### Results Directory Structure

```
Fat-Tree-Data-Center-Topology/Code/results/
├── bw_5mbit_new/
│   ├── sim.log              # Simulation parameters
│   ├── latencies.csv        # Per-batch latency data
│   ├── throughput.csv       # iperf throughput measurements
│   ├── h1_client.log        # Worker (client) logs
│   ├── h16_server.log       # Parameter server logs
│   ├── iperf_client.log     # iperf client output
│   ├── iperf_server.log     # iperf server output
│   ├── ping.log             # Ping statistics
│   └── c*_stats_*.log       # Switch port statistics
├── bw_10mbit_new/
│   └── [same structure]
├── bw_20mbit_new/
│   └── [same structure]
├── tcp_10mbit/
│   └── [same structure]
├── dctcp_10mbit/
│   └── [same structure]
└── plots/
    ├── bandwidth_vs_latency.png
    ├── bandwidth_vs_throughput.png
    ├── tcp_vs_dctcp_latency_boxplot.png
    ├── tcp_vs_dctcp_latency_cdf.png
    ├── tcp_vs_dctcp_throughput.png
    ├── latency_stats.csv
    └── throughput_stats.csv
```

### Visualization

Generate plots and statistics:

```bash
# Visualize bandwidth experiment results
python3 visualize_results.py --result-dir Fat-Tree-Data-Center-Topology/Code/results/

# Visualize CIFAR-10 training profile
python3 visualize_cifar_profile.py
```

**Generated Plots**:

1. **Bandwidth vs Latency**: Shows how increasing bandwidth reduces gradient exchange latency
2. **Bandwidth vs Throughput**: Demonstrates throughput scaling with bandwidth limits
3. **TCP vs DCTCP Latency (Boxplot)**: Compares latency distributions
4. **TCP vs DCTCP Latency (CDF)**: Cumulative distribution functions
5. **TCP vs DCTCP Throughput**: Throughput comparison

### Statistical Analysis

Summary statistics are computed:

**Latency Metrics**:
- Mean latency per batch
- Median latency
- Standard deviation
- 95th percentile latency
- Minimum and maximum latency

**Throughput Metrics**:
- Average throughput
- Peak throughput
- Throughput variability

### Expected Results

**Bandwidth Impact**:
- Higher bandwidth reduces latency
- Diminishing returns above certain threshold
- Throughput scales linearly with bandwidth (up to application limits)

**TCP vs DCTCP**:
- DCTCP reduces tail latency
- DCTCP provides more predictable performance
- DCTCP better utilizes available bandwidth in presence of congestion

## Project Structure

```
Fat-Tree-Topology-/
├── README.md                           # This file
├── README-SETUP.md                     # Original setup documentation
├── .gitignore                          # Git ignore patterns
│
├── Fat-Tree-Data-Center-Topology/      # Main simulation code
│   ├── Code/
│   │   ├── fat_tree.py                 # Fat-Tree topology implementation
│   │   ├── run_sim_fat_tree.py         # Main experiment orchestration script
│   │   ├── run_fat_tree.py             # Simple topology launcher
│   │   ├── traffic_replay.py           # Traffic replay client/server
│   │   ├── parse_latency.py            # Latency log parser
│   │   ├── run_all.sh                  # Run all experiments
│   │   ├── latencies.csv               # Parsed latency data
│   │   ├── throughput.csv              # Parsed throughput data
│   │   └── results/                    # Experiment results directory
│   │       ├── bw_5mbit_new/
│   │       ├── bw_10mbit_new/
│   │       ├── bw_20mbit_new/
│   │       ├── tcp_10mbit/
│   │       ├── dctcp_10mbit/
│   │       └── plots/
│   ├── Documentation/
│   │   └── Project Report (3).pdf      # Technical report
│   ├── LICENSE.txt
│   └── README.md                       # Original project README
│
├── cifar_profile.json                  # Detailed CIFAR-10 training profile
├── cifar_traffic_profile.csv           # Simplified traffic profile for replay
│
├── run_bandwidth_tests.sh              # Bandwidth experiment script
├── run_tcp_dctcp_compare.sh            # TCP vs DCTCP comparison script
│
├── Fat Tree.py                         # Standalone Fat-Tree implementation
├── train.py                            # PyTorch DDP training script
│
├── test_bandwidth.py                   # Bandwidth testing utilities
├── test_fattree_bw.py                  # Fat-Tree bandwidth tests
├── fix_throughput_stats.py             # Throughput data processing
│
├── visualize_results.py                # Results visualization script
├── visualize_cifar_profile.py          # Traffic profile visualization
│
└── verify_ssh_agent.sh                 # SSH agent verification (for git push)
```

### Key Files

**fat_tree.py**: Implements the Fat-Tree topology class with configurable k parameter. Creates switches, hosts, and links with proper OpenFlow 1.3 configuration.

**run_sim_fat_tree.py**: Main orchestration script that:
- Initializes Mininet with Fat-Tree topology
- Configures bandwidth and queue disciplines
- Launches measurement tools (iperf, ping)
- Runs traffic replay
- Collects and saves results

**traffic_replay.py**: Implements the gradient exchange protocol:
- Server mode: Receives gradient data and logs timestamps
- Client mode: Reads traffic profile and sends data with appropriate delays

**visualize_results.py**: Generates plots and statistical summaries from experiment results.

## Technical Implementation Details

### Switch Configuration

Switches are configured with OpenFlow 1.3:

```python
sw = self.addSwitch(
    name,
    dpid=new_dpid(),          # Unique datapath ID
    protocols='OpenFlow13'     # OpenFlow version
)
```

### Link Configuration

Links use TCLink for traffic control:

```python
self.addLink(
    switch1, switch2,
    cls=TCLink,
    bw=10,                     # Bandwidth in Mbps
    delay='0ms',               # Propagation delay
    loss=0,                    # Packet loss percentage
    max_queue_size=1000        # Queue size in packets
)
```

### Bandwidth Limiting with Token Bucket Filter

Applied to core-aggregation links:

```python
def apply_core_rate(net, rate, tbf_limit="200kb", burst="100kb"):
    for link in net.links:
        n1, n2 = link.intf1.node, link.intf2.node
        if is_core_agg_link(n1, n2):
            n1.cmd(f"tc qdisc replace dev {link.intf1.name} "
                   f"root tbf rate {rate} burst {burst} limit {tbf_limit}")
```

**TBF Parameters**:
- **rate**: Target bandwidth
- **burst**: Maximum burst size (controls burstiness)
- **limit**: Maximum queue size (controls latency)

### Network Emulation with netem

For adding delay, loss, or jitter:

```bash
tc qdisc add dev eth0 root netem delay 10ms loss 0.1%
```

### DCTCP Configuration

Requires kernel support (Linux 3.10+):

```bash
# Enable DCTCP
sysctl -w net.ipv4.tcp_congestion_control=dctcp
sysctl -w net.ipv4.tcp_ecn=1

# Configure ECN marking with RED
tc qdisc add dev eth0 root red \
    limit 1000000 \
    min 30000 \
    max 35000 \
    avpkt 1000 \
    burst 55 \
    ecn \
    adaptive
```

### Process Management in Mininet

Running background processes on hosts:

```python
# Start server in background
h16.cmd('python3 traffic_replay.py --mode server --port 5000 > server.log 2>&1 &')

# Start client and wait for completion
h1.cmd('python3 traffic_replay.py --mode client --host 10.0.0.16 --port 5000 --csv traffic.csv')
```

### Logging and Output Redirection

All processes log to separate files:

```python
h1.cmd('iperf -c 10.0.0.16 -p 5001 -t 10 > iperf_client.log 2>&1 &')
h16.cmd('iperf -s -p 5001 > iperf_server.log 2>&1 &')
```

## Troubleshooting

### Issue: Controller Connection Failed

**Symptoms**:
- Switches cannot connect to controller
- Error: "Unable to contact the remote controller"

**Solutions**:
```bash
# Check if RYU is running
ps aux | grep ryu-manager

# Check if port 6653 is listening
sudo lsof -i :6653

# Start RYU if not running
source ~/ryu/venv/bin/activate
ryu-manager ryu.app.simple_switch_13
```

### Issue: Hosts Cannot Ping Each Other

**Symptoms**:
- Ping fails between hosts
- Packet loss 100%

**Solutions**:
```bash
# In Mininet CLI, dump network info
mininet> net
mininet> links
mininet> pingall

# Check OpenFlow rules
mininet> sh ovs-ofctl -O OpenFlow13 dump-flows s1

# Restart with STP if loops exist
# ryu-manager ryu.app.simple_switch_stp_13
```

### Issue: Mininet Does Not Exit Cleanly

**Symptoms**:
- Mininet crashes or hangs
- Processes remain after exit

**Solutions**:
```bash
# Clean up Mininet
sudo mn -c

# Kill remaining processes
sudo killall -9 python3
sudo killall -9 iperf

# Remove OVS bridges
sudo ovs-vsctl list-br | xargs -L1 sudo ovs-vsctl del-br
```

### Issue: Permission Denied Errors

**Symptoms**:
- "Permission denied" when running Mininet
- Cannot create network interfaces

**Solutions**:
```bash
# Run with sudo
sudo python3 run_sim_fat_tree.py ...

# Add user to relevant groups (then logout/login)
sudo usermod -aG sudo $USER
```

### Issue: Import Errors for Mininet Modules

**Symptoms**:
- "ModuleNotFoundError: No module named 'mininet'"

**Solutions**:
```bash
# Set Python path
export PYTHONPATH=$HOME/mininet:$PYTHONPATH

# Add to .bashrc for persistence
echo 'export PYTHONPATH=$HOME/mininet:$PYTHONPATH' >> ~/.bashrc
source ~/.bashrc
```

### Issue: Traffic Replay Connection Timeout

**Symptoms**:
- Client cannot connect to server
- "Connection timed out" errors

**Solutions**:
```bash
# Verify host IPs in Mininet CLI
mininet> h1 ifconfig
mininet> h16 ifconfig

# Check if server is listening
mininet> h16 netstat -tuln | grep 5000

# Increase timeout in traffic_replay.py
sock.settimeout(60)  # Increase from 30
```

### Issue: Results Directory Not Created

**Symptoms**:
- "No such file or directory" when saving results
- Results are missing

**Solutions**:
```bash
# Create results directory manually
mkdir -p Fat-Tree-Data-Center-Topology/Code/results

# Check permissions
ls -ld Fat-Tree-Data-Center-Topology/Code/results

# Fix permissions if needed
chmod 755 Fat-Tree-Data-Center-Topology/Code/results
```

### Issue: Multiple RYU Controllers Running

**Symptoms**:
- "Address already in use" on port 6653
- Inconsistent switch behavior

**Solutions**:
```bash
# Find and kill all RYU processes
ps aux | grep ryu-manager
sudo kill <pid>

# Or kill all
sudo killall ryu-manager

# Restart single instance
ryu-manager ryu.app.simple_switch_13
```

### Issue: Low Performance or High Latency

**Symptoms**:
- Unexpectedly high latencies
- Low throughput measurements

**Possible Causes and Solutions**:

1. **VM Resource Constraints**:
   ```bash
   # Check CPU and memory usage
   top
   
   # Allocate more resources to VM
   # Edit VM settings: increase CPU cores and RAM
   ```

2. **Competing Processes**:
   ```bash
   # Check for other heavy processes
   ps aux --sort=-%cpu | head
   
   # Stop unnecessary services
   sudo systemctl stop <service>
   ```

3. **Network Configuration**:
   ```bash
   # Check for packet loss
   mininet> iperf h1 h16
   
   # Verify MTU settings
   ifconfig
   ```

### Issue: Visualization Script Errors

**Symptoms**:
- "FileNotFoundError" when running visualize_results.py
- Empty or incomplete plots

**Solutions**:
```bash
# Verify result files exist
ls -lh Fat-Tree-Data-Center-Topology/Code/results/*/latencies.csv

# Install missing Python packages
pip install matplotlib pandas numpy seaborn

# Run with explicit result directory
python3 visualize_results.py --result-dir Fat-Tree-Data-Center-Topology/Code/results/
```

## References

### Academic Papers

1. **M. Al-Fares, A. Loukissas, and A. Vahdat**, "A Scalable, Commodity Data Center Network Architecture," *ACM SIGCOMM Computer Communication Review*, vol. 38, no. 4, pp. 63-74, 2008.
   - Original Fat-Tree data center architecture paper

2. **M. Alizadeh et al.**, "Data Center TCP (DCTCP)," *ACM SIGCOMM Computer Communication Review*, vol. 40, no. 4, pp. 63-74, 2010.
   - DCTCP congestion control mechanism

3. **B. Lantz, B. Heller, and N. McKeown**, "A Network in a Laptop: Rapid Prototyping for Software-Defined Networks," *Proc. 9th ACM SIGCOMM Workshop on Hot Topics in Networks*, 2010.
   - Mininet network emulation platform

### Documentation

- **Mininet Documentation**: http://mininet.org/
- **Open vSwitch Manual**: https://docs.openvswitch.org/
- **RYU SDN Framework**: https://ryu-sdn.org/
- **OpenFlow Specification**: https://opennetworking.org/software-defined-standards/specifications/
- **Linux Traffic Control**: https://tldp.org/HOWTO/Traffic-Control-HOWTO/

### Tools and Software

- **Mininet**: http://mininet.org/download/
- **RYU Controller**: https://github.com/faucetsdn/ryu
- **Open vSwitch**: https://www.openvswitch.org/
- **iperf**: https://iperf.fr/
- **PyTorch**: https://pytorch.org/

### Related Projects

- **Reproducible Network Research**: https://reproducingnetworkresearch.wordpress.com/
- **Mininet Topology Zoo**: https://github.com/mininet/mininet/wiki/Topology-Zoo
- **SDN Hub**: http://sdnhub.org/

---

## Contact and Support

For questions, issues, or contributions:
- GitHub Issues: https://github.com/andrew45110/Fat-Tree-Topology-/issues
- Email: andrew45110@gmail.com

## License

See [LICENSE](Fat-Tree-Data-Center-Topology/LICENSE.txt) for details.

## Acknowledgments

This project builds upon the foundational work of:
- Mohammad Al-Fares and colleagues for the Fat-Tree architecture
- The Mininet team for the network emulation platform
- The RYU community for the SDN controller framework
- The Open vSwitch project for software-defined networking infrastructure

---

**Last Updated**: December 2025
