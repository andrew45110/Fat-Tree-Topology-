# Fat-Tree Data Center Network Simulation Project
## Error Report and Rectification Summary

This document details all the errors, issues, and challenges encountered during the preparation of the Fat-Tree Data Center Network Simulation project for distribution, along with the solutions implemented.

---

## Table of Contents

### Packaging & Distribution Issues
1. [Unnecessary Large Files Included in Initial Zip](#1-unnecessary-large-files-included-in-initial-zip)
2. [Missing RYU Virtual Environment Activation Step](#2-missing-ryu-virtual-environment-activation-step)
3. [Missing Mininet Cleanup Instructions](#3-missing-mininet-cleanup-instructions)
4. [Multiple RYU Controller Instances Conflict](#4-multiple-ryu-controller-instances-conflict)
5. [Zip Process Appeared to Hang](#5-zip-process-appeared-to-hang)
6. [Missing README Documentation in Zip File](#6-missing-readme-documentation-in-zip-file)
7. [Missing Helper Scripts in Zip File](#7-missing-helper-scripts-in-zip-file)
8. [Hardcoded File Paths in Scripts](#8-hardcoded-file-paths-in-scripts)
9. [Missing TCP vs DCTCP Comparison Script](#9-missing-tcp-vs-dctcp-comparison-script)
10. [Unclear Experiment Results Location](#10-unclear-experiment-results-location)

### SDN-Specific Errors
11. [Ryu Controller Startup Warnings](#11-ryu-controller-startup-warnings)
12. [Pingall Failures / Destination Host Unreachable](#12-pingall-failures--destination-host-unreachable)
13. [Controller-Switch Connectivity Issues](#13-controller-switch-connectivity-issues)
14. [Fat-Tree Topology Complexity Issues](#14-fat-tree-topology-complexity-issues)
15. [Flow Table Verification](#15-flow-table-verification)

---

## 1. Unnecessary Large Files Included in Initial Zip

### Problem
The initial attempt to zip the entire mininet home directory included the PyTorch virtual environment (`torch-env/`) which was approximately **5.1 GB** in size. This was unnecessary because:
- The project had transitioned from actual AI training to traffic simulation using `traffic_replay.py`
- The traffic simulation only requires the `cifar_traffic_profile.csv` file, not the actual PyTorch installation or CIFAR-10 dataset

### Files That Were Unnecessarily Large
| File/Directory | Size | Purpose |
|----------------|------|---------|
| `torch-env/` | 5.1 GB | PyTorch virtual environment |
| `data/cifar-10-python.tar.gz` | 170 MB | CIFAR-10 dataset archive |
| `mininet_home_backup.zip` | 113 MB | Old backup file |

### Solution
Removed the unnecessary files to free up approximately 5.4 GB of disk space:
```bash
rm -rf torch-env/ data/cifar-10-python.tar.gz mininet_home_backup.zip
```

### Verification
After removal, disk usage dropped significantly:
```bash
df -h .
# Result: 51GB available (previously much less)
```

---

## 2. Missing RYU Virtual Environment Activation Step

### Problem
The initial documentation did not include the critical step of activating the RYU virtual environment before starting the controller. Without this step, users would encounter import errors when trying to run the RYU controller.

### Error Scenario
If a user tried to run the RYU controller without activating the virtual environment:
```bash
cd ryu
./run_ryu.py ryu.app.simple_switch_13
# Would fail with ModuleNotFoundError for eventlet or ryu modules
```

### Solution
Updated the README-SETUP.md to include the virtual environment activation step:
```bash
# First, activate the RYU virtual environment:
source ~/ryu/venv/bin/activate

# Then start the RYU controller:
cd ryu
./run_ryu.py ryu.app.simple_switch_13
```

---

## 3. Missing Mininet Cleanup Instructions

### Problem
If a user ran `run_sim_fat_tree.py` and then tried to run it again without cleaning up, they would encounter errors because:
- Previous Mininet instances might still be running
- Network interfaces from the previous run might still exist
- Open vSwitch might have stale configurations

### Error Scenario
```bash
# Running simulation without cleanup
sudo python3 run_sim_fat_tree.py ...
# Error: RTNETLINK answers: File exists
# Error: could not find host interface
```

### Solution
Added instructions to always run cleanup before starting a new simulation:
```bash
sudo mn -c
```

This command:
- Removes all Mininet-created network interfaces
- Cleans up Open vSwitch configurations
- Kills any lingering Mininet processes

---

## 4. Multiple RYU Controller Instances Conflict

### Problem
If multiple RYU controller instances are running simultaneously, they compete for the same OpenFlow port (6653), causing:
- Connection failures
- Unpredictable behavior
- Switches connecting to the wrong controller

### Error Scenario
```bash
# Second RYU instance fails to bind to port
OSError: [Errno 98] Address already in use
```

### Solution
Added instructions to check for and kill existing RYU processes before starting a new one:

```bash
# Check for running RYU processes
ps aux | grep ryu

# Kill existing RYU processes if found
sudo kill $(ps aux | grep ryu | grep -v grep | awk '{print $2}')

# Alternatively, check if port 6653 is in use
sudo lsof -i :6653
```

---

## 5. Zip Process Appeared to Hang

### Problem
When creating the zip file, the process appeared to "hang" or stop mid-execution. The terminal output was truncated, making it seem like the zip command had failed.

### Initial Concern
```bash
cd mininet_fat_tree_project && zip -r ../mininet_fat_tree_minimal.zip *
# Output stopped after showing partial file list
# Command appeared interrupted
```

### Investigation
Checked if the zip process was still running:
```bash
ps aux | grep -i zip
# No active zip process found
```

Checked if the system killed the process due to memory issues:
```bash
dmesg | grep -i -e killed -e oom | tail -10
# No OOM killer messages found
```

### Actual Cause
The zip process had actually **completed successfully**. The apparent "hang" was due to:
1. The large number of files being processed generated extensive output
2. The terminal output was truncated for display purposes
3. The zip file was successfully created

### Verification
```bash
ls -lah mininet_fat_tree_minimal.zip
# Result: 41M - file exists and has proper size

unzip -t mininet_fat_tree_minimal.zip | tail
# Result: "No errors detected in compressed data"
```

### Lesson Learned
Always verify zip file creation by:
1. Checking if the file exists and has a reasonable size
2. Running `unzip -t <file>` to test integrity

---

## 6. Missing README Documentation in Zip File

### Problem
The comprehensive README-SETUP.md that was created with all the setup and troubleshooting instructions was not included in the initial zip file.

### Discovery
```bash
unzip -l mininet_project_essential.zip | grep -i readme
# Only showed READMEs from subdirectories, not the main setup README
```

### Solution
Added the README to the zip file in two forms:
```bash
# Add as README-SETUP.md
zip -u mininet_project_essential.zip README-SETUP.md

# Also add as README.md for visibility
cp README-SETUP.md README.md
zip -u mininet_project_essential.zip README.md
rm README.md
```

---

## 7. Missing Helper Scripts in Zip File

### Problem
The `fix_paths.sh` and `setup.sh` helper scripts were created but not included in the initial `mininet_project_essential.zip` file.

### Scripts and Their Purposes
| Script | Purpose |
|--------|---------|
| `fix_paths.sh` | Updates hardcoded paths to match installation location |
| `setup.sh` | Creates RYU virtual environment and installs dependencies |

### Solution
Added both scripts to the zip file:
```bash
# Extract, add scripts, and update zip
mkdir -p test_extract && cd test_extract
unzip -q ../mininet_project_essential.zip
cp ../mininet_fat_tree_project/fix_paths.sh .
cp ../mininet_fat_tree_project/setup.sh .
chmod +x fix_paths.sh setup.sh
zip -u ../mininet_project_essential.zip fix_paths.sh setup.sh
```

---

## 8. Hardcoded File Paths in Scripts

### Problem
Several scripts contained hardcoded paths that assumed the project was installed in `/home/mininet/`. This would cause failures when users installed the project in different locations.

### Affected Files and Hardcoded Paths
| File | Hardcoded Path |
|------|----------------|
| `run_bandwidth_tests.sh` | `/home/mininet/cifar_traffic_profile.csv` |
| `run_tcp_dctcp_compare.sh` | `/home/mininet/cifar_traffic_profile.csv` |
| `visualize_cifar_profile.py` | `/home/mininet/cifar_traffic_profile.csv` |

### Error Scenario
```bash
./run_bandwidth_tests.sh
# Error: FileNotFoundError: /home/mininet/cifar_traffic_profile.csv not found
```

### Solution
Created `fix_paths.sh` script that automatically updates paths based on installation location:

```bash
#!/bin/bash
echo "Fixing paths in project files..."
CURR_DIR=$(pwd)

# Fix paths in run_bandwidth_tests.sh
sed -i "s|/home/mininet/cifar_traffic_profile.csv|$CURR_DIR/cifar_traffic_profile.csv|g" run_bandwidth_tests.sh

# Fix paths in run_tcp_dctcp_compare.sh
sed -i "s|/home/mininet/cifar_traffic_profile.csv|$CURR_DIR/cifar_traffic_profile.csv|g" run_tcp_dctcp_compare.sh

# Fix paths in visualize_cifar_profile.py
sed -i "s|/home/mininet/cifar_traffic_profile.csv|$CURR_DIR/cifar_traffic_profile.csv|g" visualize_cifar_profile.py

echo "Path fixing complete!"
```

---

## 9. Missing TCP vs DCTCP Comparison Script

### Problem
The `run_tcp_dctcp_compare.sh` script, which is essential for comparing TCP and DCTCP performance, was not included in the zip file.

### Discovery
```bash
unzip -l mininet_project_essential.zip | grep -E "(run_tcp_dctcp_compare|run_bandwidth_tests).sh"
# Only showed run_bandwidth_tests.sh, missing run_tcp_dctcp_compare.sh
```

### Complication
The script was located in an unusual path due to a tilde character in the directory name:
```bash
# Incorrect path (didn't work)
~/Fat-Tree-Data-Center-Topology/Code/run_tcp_dctcp_compare.sh

# Correct path (note the escaped tilde)
~/~/Fat-Tree-Data-Center-Topology/Code/run_tcp_dctcp_compare.sh
```

### Solution
```bash
cp ~/\~/Fat-Tree-Data-Center-Topology/Code/run_tcp_dctcp_compare.sh .
chmod +x run_tcp_dctcp_compare.sh
zip -u mininet_project_essential.zip run_tcp_dctcp_compare.sh
```

---

## 10. Unclear Experiment Results Location

### Problem
The initial documentation did not clearly explain where experiment results were stored and which files contained meaningful data versus which were empty or incomplete.

### Issue
Some result directories contained files with minimal or no data:
```bash
ls -lh Fat-Tree-Data-Center-Topology/Code/results/bw_*/latencies.csv
# Some files were only 17 bytes (just headers, no data)
# Others were 1.4K (actual data)
```

### Solution
Updated README-SETUP.md with detailed explanation of:

1. **Results Directory Structure:**
```
Fat-Tree-Data-Center-Topology/Code/results/
├── bw_5mbit_new/      # Results for 5Mbps bandwidth test
│   ├── latencies.csv  # Contains batch number and latency in seconds
│   ├── throughput.csv # Contains throughput measurements in Mbps
│   └── various log files
├── tcp_10mbit/        # Results for TCP with 10Mbps bandwidth
├── dctcp_10mbit/      # Results for DCTCP with 10Mbps bandwidth
└── plots/             # Generated visualization plots
    ├── latency_stats.csv    # Summary statistics
    └── throughput_stats.csv # Throughput summary
```

2. **Which directories contain complete data:**
   - `bw_5mbit_new/`
   - `tcp_10mbit/`
   - `dctcp_10mbit/`

3. **How visualization script uses these files:**
   - Explained that `visualize_results.py` reads from specific directories
   - Pointed users to the `plots/` directory for summary statistics

---

## Summary of All Files Added to Final Zip

| File | Purpose | Status |
|------|---------|--------|
| `README.md` | Main documentation | Added |
| `README-SETUP.md` | Detailed setup guide | Added |
| `fix_paths.sh` | Path configuration script | Added |
| `setup.sh` | Environment setup script | Added |
| `run_bandwidth_tests.sh` | Bandwidth experiment script | Already present |
| `run_tcp_dctcp_compare.sh` | TCP/DCTCP comparison script | Added |
| `cifar_traffic_profile.csv` | Traffic pattern data | Already present |
| `cifar_profile.json` | Profile data | Already present |
| `visualize_results.py` | Results visualization | Already present |
| `visualize_cifar_profile.py` | CIFAR profile visualization | Already present |
| `Fat-Tree-Data-Center-Topology/` | Main project code | Already present |
| `ryu/` | RYU controller | Already present |

---

## Final Verification Checklist

Before distribution, the following checks were performed:

- [x] Zip file integrity verified with `unzip -t`
- [x] All essential scripts included
- [x] README documentation included
- [x] Helper scripts (fix_paths.sh, setup.sh) included
- [x] TCP/DCTCP comparison script included
- [x] Unnecessary large files (torch-env, CIFAR dataset) removed
- [x] All scripts have execute permissions

---

## Installation Instructions for End Users

1. Extract the zip file in the Mininet VM's home directory:
   ```bash
   cd ~
   unzip mininet_project_essential.zip
   ```

2. Run the path fixing script:
   ```bash
   ./fix_paths.sh
   ```

3. Run the setup script:
   ```bash
   ./setup.sh
   ```

4. Follow the instructions in README.md to run experiments

---

## SDN-Specific Errors and Solutions

This section documents the core SDN networking issues encountered during development and testing of the Fat-Tree topology with the Ryu controller.

---

### 11. Ryu Controller Startup Warnings

#### Errors Encountered

Two persistent warnings appeared every time the Ryu controller was launched:

1. **`RuntimeWarning: greenlet.greenlet size changed, may indicate binary incompatibility`**
   - Caused by ABI mismatch between different greenlet builds across system paths

2. **`X RLock(s) were not greened`**
   - Caused by Eventlet monkey patching occurring too late in the import sequence

#### Attempted Fixes (Unsuccessful)

| Attempt | Why It Failed |
|---------|---------------|
| Reinstalled greenlet with `pip3 install --force-reinstall --no-binary greenlet` pinned to v1.1.3 | Python still loaded stray `.so` files from other paths |
| Purged greenlet from `~/.local` and `/usr/local` | Multiple copies persisted; prefix confusion |
| Synchronized Python prefixes | Ryu's `manager.py` unconditionally imports `gevent.monkey`, pulling in vendored `_greenlet*.so` |

#### Final Solution: Virtual Environment + Custom Launcher

**Step 1: Create a clean virtual environment**

```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip setuptools wheel
pip install 'setuptools<58'  # Required for Ryu's PBR hooks
pip install -e .              # Install Ryu in editable mode
```

**Step 2: Create `run_ryu.py` launcher script**

This script ensures eventlet is monkey-patched *before* any other imports:

```python
#!/usr/bin/env python3
import os
os.environ['RYU_USE_GEVENT'] = '0'

import eventlet
eventlet.monkey_patch()

from ryu.cmd.manager import main
main()
```

**Step 3: Run with the launcher**

```bash
chmod +x run_ryu.py
sudo ./run_ryu.py --ofp-tcp-listen-port 6653 ryu.app.simple_switch_stp_13
```

**Why It Worked:**
- The venv isolated a single, consistent greenlet build
- The wrapper ensured eventlet patching happened before any locks were created
- Gevent was installed in the venv so Ryu's imports never failed

---

### 12. Pingall Failures / Destination Host Unreachable

#### Symptoms

- `pingall` command failed with "Destination Host Unreachable"
- Hosts could not communicate across the Fat-Tree topology
- Some pings worked, others failed inconsistently

#### Root Cause Analysis

**ARP Resolution Failure:**

When checking ARP tables on hosts:

```bash
mininet> h3 arp -n
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.0.4                         (incomplete)                              h3-eth0

mininet> h4 arp -n
Address                  HWtype  HWaddress           Flags Mask            Iface
10.0.0.3                 ether   00:00:00:00:00:03   C                     h4-eth0
```

**Interpretation:**
- h3's ARP entry for h4 (10.0.0.4) shows `(incomplete)` — h3 sent an ARP request but never received a reply
- h4 has a complete ARP entry for h3 — the reverse direction worked
- Without h4's MAC address, h3 cannot form valid unicast frames
- The Ryu controller never sees unicast traffic to trigger flow installation

#### Why This Matters for SDN

1. **No Unicast Flows Installed:** The learning switch logic depends on seeing unicast packets. If ARP fails, no unicast packets are generated.

2. **Broadcast Handling:** Simple learning switches often don't install specific flows for broadcast traffic (like ARP requests). If ARP replies are lost, the whole communication chain breaks.

3. **Timing Issues:** Even if flows are installed, they may expire before verification if idle timeouts are short.

#### Debugging Steps

```bash
# Check ARP tables on hosts
mininet> h3 arp -n
mininet> h4 arp -n

# Run tcpdump to observe ARP traffic
mininet> h4 tcpdump -i h4-eth0 arp &

# Inspect flow tables on switches
sudo ovs-ofctl dump-flows s1 -O OpenFlow13
```

#### Solution

1. **Use STP-enabled learning switch** to prevent broadcast storms in the Fat-Tree topology:
   ```bash
   sudo ./run_ryu.py --ofp-tcp-listen-port 6653 ryu.app.simple_switch_stp_13
   ```

2. **Verify controller-switch connectivity** before testing:
   ```bash
   sudo ovs-vsctl show
   # Look for "is_connected: true" on each switch
   ```

3. **Clean up stale state** before each test:
   ```bash
   sudo mn -c
   ```

---

### 13. Controller-Switch Connectivity Issues

#### Potential Issues

- Port number mismatch between Ryu controller and OVS switches
- Firewall blocking OpenFlow port (6633 or 6653)
- Service conflicts on the OpenFlow port

#### Verification Command

```bash
sudo ovs-vsctl show
```

**Expected Output (Good):**
```
Bridge "s1"
    Controller "tcp:127.0.0.1:6653"
        is_connected: true
    ...
```

If `is_connected: true` appears for all switches, the TCP connection is established and there are no firewall/port issues.

#### Troubleshooting Connectivity

```bash
# Check if controller port is in use
sudo lsof -i :6653

# Check for existing Ryu processes
ps aux | grep ryu

# Kill stale Ryu processes
sudo kill $(ps aux | grep ryu | grep -v grep | awk '{print $2}')
```

---

### 14. Fat-Tree Topology Complexity Issues

#### Problem

The custom Fat-Tree topology with core, aggregation, and edge switches is significantly more complex than simple linear or tree topologies. Issues encountered:

- Cross-pod connectivity failures
- Loops causing broadcast storms
- Inconsistent host placement

#### Key Considerations

| Layer | Switch Type | Connectivity |
|-------|-------------|--------------|
| Core | Core switches | Connect to all pods |
| Aggregation | Aggregation switches | Connect core to edge |
| Edge | Edge switches | Connect directly to hosts |

#### Solution

1. **Use STP (Spanning Tree Protocol)** to prevent loops:
   ```bash
   ryu.app.simple_switch_stp_13  # Instead of simple_switch_13
   ```

2. **Verify link establishment** between all layers

3. **Check host IP/MAC assignment** consistency

4. **Use OpenFlow13 protocol** explicitly:
   ```python
   # In topology code
   switch = self.addSwitch('s1', protocols='OpenFlow13')
   ```

---

### 15. Flow Table Verification

#### How to Check Flow Rules

```bash
# Dump flows on a specific switch
sudo ovs-ofctl dump-flows s1 -O OpenFlow13

# Check all switches in a loop
for i in 1 2 3 4; do
    echo "=== Switch s$i ==="
    sudo ovs-ofctl dump-flows s$i -O OpenFlow13
done
```

#### Expected Flow Entries

| Entry Type | Description |
|------------|-------------|
| Default rule (priority=0) | Forwards unmatched packets to controller (packet-in) |
| Learned rules (priority>0) | Match on MAC addresses, forward to specific port |

**Signs of Successful Learning:**
- Non-zero `n_packets` and `n_bytes` counters on learned flows
- Flows with specific `dl_dst` (destination MAC) matches
- Multiple flow entries beyond just the default table-miss rule

---

### SDN Error Summary Table

| Error | Root Cause | Solution |
|-------|-----------|----------|
| Greenlet ABI warning | Multiple conflicting greenlet builds | Clean venv with single build |
| RLock not greened | Eventlet patched too late | Custom launcher with early `monkey_patch()` |
| Incomplete ARP entries | ARP replies not reaching requester | Use STP-enabled switch, verify paths |
| Destination unreachable | No unicast flows installed | Resolve ARP first → flows auto-install |
| Switch not connected | Port mismatch / firewall | Verify with `ovs-vsctl show` |
| Broadcast storms | Loops in Fat-Tree topology | Use `simple_switch_stp_13` |

---

*Report generated during project preparation session*
*Date: April 30, 2025*
*Updated: December 9, 2025 - Added SDN-specific troubleshooting section*

