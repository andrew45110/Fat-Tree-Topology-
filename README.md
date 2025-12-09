# Fat-Tree Data Center Network Simulation

A comprehensive network simulation project that emulates a Fat-Tree data center topology using Mininet to study distributed machine learning traffic patterns, specifically CIFAR-10 gradient exchange performance under various network conditions.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Experiments](#experiments)
- [Results](#results)
- [Troubleshooting](#troubleshooting)
- [License](#license)

## 🎯 Overview

This project simulates a three-tier Fat-Tree data center network topology to analyze how network conditions affect distributed machine learning workloads. The simulation:

- **Emulates** a k-port Fat-Tree topology (k=4 by default) with 20 switches and 16 hosts
- **Replays** real CIFAR-10 training traffic patterns to simulate gradient exchange
- **Measures** latency, throughput, and network performance under various conditions
- **Compares** TCP vs DCTCP (Data Center TCP) congestion control mechanisms
- **Analyzes** the impact of bandwidth constraints on distributed ML training

The Fat-Tree topology, proposed by M. Al-Fares, is a cost-effective data center architecture that provides full bisection bandwidth using commodity Ethernet switches.

## ✨ Features

- **Fat-Tree Topology**: Three-tier architecture (core, aggregation, edge layers)
- **SDN Support**: OpenFlow 1.3 compatible with RYU controller
- **Traffic Replay**: Realistic CIFAR-10 gradient exchange traffic patterns
- **Bandwidth Testing**: Automated experiments with configurable bandwidth limits (5, 10, 20 Mbps)
- **Protocol Comparison**: TCP vs DCTCP performance analysis
- **Comprehensive Metrics**: Latency, throughput, RTT, and jitter measurements
- **Visualization Tools**: Automated plotting and statistical analysis
- **Distributed Training**: Support for PyTorch DDP training over the network

## 📦 Prerequisites

- **OS**: Ubuntu 18.04/20.04 (or compatible Linux distribution)
- **Mininet**: Network emulator for creating virtual network topologies
- **RYU Controller**: OpenFlow SDN controller
- **Python**: 3.x with required packages
- **Network Tools**: iperf, ping, tc (traffic control)

### Required Python Packages

```bash
pip install torch torchvision numpy matplotlib pandas
```

## 🚀 Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/andrew45110/Fat-Tree-Topology-.git
   cd Fat-Tree-Topology-
   ```

2. **Install Mininet** (if not already installed):
   ```bash
   git clone https://github.com/mininet/mininet.git
   cd mininet
   sudo ./util/install.sh -a
   ```

3. **Install RYU Controller**:
   ```bash
   git clone https://github.com/faucetsdn/ryu.git
   cd ryu
   python3 -m venv venv
   source venv/bin/activate
   pip install -r tools/pip-requires
   ```

4. **Set up Python path**:
   ```bash
   export PYTHONPATH=$HOME/mininet:$PYTHONPATH
   ```

## 🏃 Quick Start

### 1. Clean Up Previous Instances

Before starting a new simulation, clean up any previous Mininet instances:

```bash
sudo mn -c
```

### 2. Start the RYU Controller

In a separate terminal, start the RYU controller:

```bash
source ~/ryu/venv/bin/activate
cd ryu
./run_ryu.py ryu.app.simple_switch_13
```

Or with spanning tree protocol:

```bash
./run_ryu.py ryu.app.simple_switch_stp_13
```

**Keep this terminal open** with the controller running.

### 3. Run a Quick Test

Run a bandwidth test:

```bash
./run_bandwidth_tests.sh
```

## 📖 Usage

### Running Bandwidth Experiments

The `run_bandwidth_tests.sh` script runs experiments with different bandwidth constraints (5, 10, 20 Mbps):

```bash
./run_bandwidth_tests.sh
```

This script:
- Runs tests sequentially with 5, 10, and 20 Mbps bandwidth limits
- Measures latency of each gradient exchange
- Measures throughput using iperf
- Collects results in separate directories for each bandwidth setting

### TCP vs DCTCP Comparison

Compare standard TCP with Data Center TCP (DCTCP) performance:

```bash
./run_tcp_dctcp_compare.sh
```

This script:
- Runs a simulation using standard TCP and collects performance data
- Runs an identical simulation using DCTCP with ECN (Explicit Congestion Notification)
- Stores results in separate directories for comparison
- Analyzes how DCTCP's congestion control affects gradient exchange

### Running Individual Tests

Run a single test with specific parameters:

```bash
sudo PYTHONPATH=$HOME/mininet python3 Fat-Tree-Data-Center-Topology/Code/run_sim_fat_tree.py \
  --k 4 \
  --ps-host h16 \
  --worker-host h1 \
  --csv cifar_traffic_profile.csv \
  --port 5000 \
  --iperf-port 5001 \
  --iperf-duration 10 \
  --core-bw 10mbit \
  --debug \
  --result-dir results/my_custom_test
```

#### Available Parameters

- `--k`: Number of ports per switch (default: 4)
- `--ps-host`: Parameter server hostname (default: h16)
- `--worker-host`: Worker hostname (default: h1)
- `--csv`: Path to traffic profile CSV file
- `--core-bw`: Core link bandwidth (e.g., 10mbit, 20mbit)
- `--qdisc`: Queue discipline (fifo, tbf, netem, dctcp)
- `--ecn`: Enable Explicit Congestion Notification
- `--auto-exit`: Skip CLI and auto-tear down after replay
- `--result-dir`: Directory to store results
- `--debug`: Enable verbose debugging output

### Visualizing Results

After running tests, visualize the results:

```bash
python3 visualize_results.py --result-dir results/
```

For CIFAR profile visualization:

```bash
python3 visualize_cifar_profile.py
```

## 📁 Project Structure

```
.
├── Fat-Tree-Data-Center-Topology/    # Main project code
│   ├── Code/                         # Simulation scripts
│   │   ├── fat_tree.py              # Fat-tree topology implementation
│   │   ├── run_sim_fat_tree.py      # Main simulation script
│   │   ├── traffic_replay.py        # Traffic replay client/server
│   │   ├── parse_latency.py         # Latency data parser
│   │   └── run_tcp_dctcp_compare.sh # TCP vs DCTCP comparison script
│   ├── Documentation/               # Project documentation
│   └── README.md                    # Original project README
├── ryu/                              # RYU OpenFlow controller
├── mininet/                          # Mininet network emulator
├── cifar_profile.json                # CIFAR-10 training timing data
├── cifar_traffic_profile.csv         # Traffic pattern CSV
├── run_bandwidth_tests.sh            # Bandwidth experiment script
├── run_tcp_dctcp_compare.sh          # TCP vs DCTCP comparison script
├── train.py                          # PyTorch DDP training script
├── visualize_results.py             # Results visualization script
├── visualize_cifar_profile.py       # CIFAR profile visualization
├── test_bandwidth.py                 # Bandwidth testing utilities
├── test_fattree_bw.py                # Fat-tree bandwidth tests
├── fix_throughput_stats.py           # Throughput statistics fixer
├── Fat Tree.py                       # Simple Fat-Tree topology script
└── README.md                         # This file
```

## 🔬 Experiments

### Experiment Types

1. **Bandwidth Experiments**: Test performance under different bandwidth constraints (5, 10, 20 Mbps)
2. **Protocol Comparison**: Compare TCP vs DCTCP performance
3. **Custom Configurations**: Run experiments with custom parameters

### Topology Details

For k=4 (default configuration):
- **Pods**: 4
- **Core Switches**: 4
- **Aggregation Switches**: 8 (2 per pod)
- **Edge Switches**: 8 (2 per pod)
- **Hosts**: 16 (4 per pod, 2 per edge switch)

## 📊 Results

### Results Directory Structure

```
Fat-Tree-Data-Center-Topology/Code/results/
├── bw_5mbit_new/      # Results for 5Mbps bandwidth test
│   ├── latencies.csv  # Batch number and latency in seconds
│   ├── throughput.csv # Throughput measurements in Mbps
│   └── *.log          # Various log files
├── bw_10mbit_new/     # Results for 10Mbps bandwidth test
├── bw_20mbit_new/     # Results for 20Mbps bandwidth test
├── tcp_10mbit/        # Results for TCP with 10Mbps bandwidth
├── dctcp_10mbit/      # Results for DCTCP with 10Mbps bandwidth
└── plots/             # Generated visualization plots
    ├── bandwidth_vs_latency.png
    ├── bandwidth_vs_throughput.png
    ├── tcp_vs_dctcp_latency_boxplot.png
    ├── tcp_vs_dctcp_latency_cdf.png
    ├── tcp_vs_dctcp_throughput.png
    ├── latency_stats.csv    # Summary statistics for latency
    └── throughput_stats.csv # Summary statistics for throughput
```

### Key Metrics

- **Latency**: Per-batch gradient exchange latency (seconds)
- **Throughput**: Network throughput measured via iperf (Mbps)
- **RTT**: Round-trip time between hosts
- **Jitter**: Variation in packet delay
- **Bisection Bandwidth**: Maximum bandwidth between two halves of the network

## 🔧 Troubleshooting

### Controller Connection Issues

If hosts can't ping each other:
- Ensure the RYU controller is running and accessible
- Check that port 6653 (OpenFlow) is not blocked
- Verify controller is using OpenFlow 1.3

```bash
# Check if controller is running
ps aux | grep ryu

# Check if port is in use
sudo lsof -i :6653
```

### Permission Issues

Many commands require sudo privileges due to network operations:

```bash
# Always use sudo for Mininet commands
sudo mn -c
sudo python3 run_sim_fat_tree.py ...
```

### Path Issues

If you get import errors, ensure PYTHONPATH includes the mininet directory:

```bash
export PYTHONPATH=$HOME/mininet:$PYTHONPATH
```

### Clean Up

If Mininet doesn't exit cleanly:

```bash
sudo mn -c
```

### Multiple Controllers

If you experience connectivity issues, check for multiple RYU controllers:

```bash
# Kill all RYU processes
sudo kill $(ps aux | grep ryu | grep -v grep | awk '{print $2}')
```

### Network Connectivity

If the simulation hangs or hosts can't communicate:
- Verify the RYU controller is running before starting Mininet
- Check that all switches are connected to the controller
- Ensure no firewall is blocking OpenFlow traffic

## 📝 Notes

- The simulation uses real CIFAR-10 training traffic patterns from actual distributed training runs
- All switches are configured with OpenFlow 1.3 protocol
- The topology supports full bisection bandwidth (1:1 oversubscription ratio)
- Results are automatically saved to CSV files for further analysis

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## 📄 License

See the [LICENSE](Fat-Tree-Data-Center-Topology/LICENSE.txt) file for details.

## 🙏 Acknowledgments

- **Fat-Tree Topology**: Based on the work by M. Al-Fares et al. ("A Scalable, Commodity Data Center Network Architecture")
- **Mininet**: Network emulator for SDN research
- **RYU**: OpenFlow SDN controller framework
- **CIFAR-10**: Dataset used for traffic pattern generation

## 📧 Contact

For questions or issues, please open an issue on GitHub or contact the maintainer.

---

**Note**: This project is designed for research and educational purposes in network simulation and distributed systems.

