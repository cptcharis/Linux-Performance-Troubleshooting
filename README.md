# Linux I/O Performance Troubleshooting Notebook

Comprehensive guide to diagnosing and resolving disk I/O bottlenecks

## What is I/O Performance Troubleshooting?


**I/O Performance Issues** = Slow disk read/write operations causing system
slowdowns \
**Symptoms :** Slow find commands, slow file copies, sluggish applications \
**Root Cause :** Disk bottleneck preventing efficient data access \
**Goal :** Identify which disk and which process is consuming I/O resources \
**Tools :** top, iostat, iotop \
**Benefit :** Pinpoint problematic processes and resolve performance issues

## Installation 

Install iostat (part of sysstat)
```
sudo apt install sysstat
```
Install iotop
```
sudo apt install iotop
```

## Step 1: Identify I/O Bottleneck with top

### Check Overall System Resource Usage


```
top
```
**What to Look For** :


| Metric | Normal Range | Warning Sign |
|--------|--------------|--------------|
| %CPU | < 80% | High CPU usage |
| %wa (I/O Wait) | < 10-20% | > 20% = I/O bottleneck |

**Key Insight** : If %wa (I/O wait percentage) is consistently high, the CPU is
spending too much time waiting for disk operations to complete.

### Understanding %wa (I/O wait)


**%wa** = Percentage of time CPU is idle while waiting for I/O operations to
finish \
**High %wa** (> 20%) = Disk is the bottleneck, not CPU \
**Low %wa** (< 10%) = Disk performance is acceptable 

## Step 2: Identify Which Disk is Slow with iostat

### List All Disks

```
lsblk
```
**Sample Output** :


| NAME | MAJ:MIN | RM | SIZE | RO | TYPE | MOUNTPOINT |
|------|---------|----|------|----|------|------------|
| xvda | 202:0 | 0 | 8G | 0 | disk | |
| xvdf | 202:80 | 0 | 8G | 0 | disk | |

### Monitor Disk I/O Activity

```
iostat -h -x -m 1 4
```
**Flags Explained** :


| Flag | Purpose |
|------|---------|
| -h | Human-readable output |
| -x | Extended statistics (detailed metrics) |
| -m | Display in megabytes per second (default: kilobytes) |
| 1 | Report interval (1 second) |
| 4 | Number of iterations |

### Sample iostat Output


| Device | r/s | w/s | rMB/s | wMB/s | %util |
|--------|-----|-----|-------|-------|-------|
| xvda | 5 | 10 | 0.5 | 1.2 | 15% |
| xvdf | 150 | 200 | 45 | 60 | 100% |


**Interpretation** :

**xvda** : Low utilization (15%) — healthy \
**xvdf : 100% utilization —** this disk is the bottleneck!

### Key Key iostat Metrics Metrics

| Metric | Meaning | What It Indicates |
|--------|---------|-------------------|
| r/s | Reads per second | Read request frequency |
| w/s | Writes per second | Write request frequency |
| rMB/s | Read throughput (MB/s) | Data read speed |
| wMB/s | Write throughput (MB/s) | Data write speed |
| await | Average I/O wait time (ms) | How long requests take |
| %util | Disk utilization | 100% = saturated/bottleneck |

## Step 3: Identify Which Process is Using I/O with iotop

### Monitor I/O by Process

```
iotop -o
```
**Flags Explained** :

| Flag | Purpose |
|------|---------|
| -o | Only show processes doing I/O (cleaner output) |

### Sample iotop Output


| PID | USER | DISK READ | DISK WRITE | COMMAND |
|-----|------|-----------|-----------|---------|
| 1234 | root | 45.2 MB/s | 60.1 MB/s | dd if=/dev/zero of=/tmp/test |
| 5678 | user | 2.1 MB/s | 0.5 MB/s | find / -type f |
| 9999 | root | 0.3 MB/s | 0.1 MB/s | |


# Linux System Activity Reporter (SAR) Monitoring Notebook

Comprehensive guide to performance monitoring and analysis

## What is SAR? 

**SAR** = System Activity Reporter \
**Package :** Part of sysstat \
**Purpose :** Collect, report, and save system activity information \
**Key Feature :** Historical performance data analysis — look back at past
system behavior \
**Scope :** Monitor CPU, memory, disk I/O, and network activity \
**Benefit :** Identify system bottlenecks (high CPU, slow disk I/O, etc.)

## Installation 

```
sudo apt install sysstat
```
## CPU Monitoring 

### Basic CPU Usage Report


```
sar -u 15
```
**Purpose** : Display CPU usage every 1 second for 15 iterations.

### Monitor CPU Every 2 Seconds (3 Iterations)

```
sar -u 2 3
```
**Sample Output** :

| Time | %user | %nice | %system | %iowait | %steal | %idle |
|------|-------|-------|---------|---------|--------|-------|
| 10:00:01 | 15.2 | 0.0 | 8.5 | 5.3 | 0.0 | 70.0 |
| 10:00:03 | 18.1 | 0.0 | 9.2 | 4.8 | 0.0 | 68.0 |
| 10:00:05 | 16.5 | 0.0 | 8.9 | 5.1 | 0.0 | 69.0 |
| Average | 16.6 | 0.0 | 8.9 | 5.1 | 0.0 | 69.0 |


### Key CPU Metrics

| Metric | Meaning | What It Indicates |
|--------|---------|-------------------|
| %user | User process CPU | Applications and programs you run |
| %system | Kernel/system CPU | Core OS functions |
| %iowait | I/O wait time | CPU waiting for disk/network operations |
| %idle | Idle CPU | CPU not being used |


### Advanced CPU Monitoring Commands

Per-CPU statistics (all cores) 
```
sar -P ALL 2 3 
```
CPU utilization by process
```
sar -x 2 3     
```
CPU frequency monitoring 
```
sar -m CPU 2 3 
```
Context switches and interrupts
```
sar -w 2 3     
```

## Memory Monitoring 

### Monitor Memory Usage (2 Seconds, 3 Iterations)

```
sar -r 2 3
```
**Sample Output** :

| Time | kbmemfree | kbmemused | %memused | kbbuffers | kbcached | kbcommit |
|------|-----------|-----------|----------|-----------|----------|----------|
| 10:00:01 | 2,048,576 | 1,048,576 | 33.8 | 256,000 | 512,000 | 1,500,000 |

### Key Memory Metrics

| Metric | Meaning |
|--------|---------|
| kbmemfree | Free memory (kilobytes) |
| %memused | Percentage of total memory in use |
| kbcached | Memory used for file system caching (improves performance) |
| kbcommit | Memory needed for current workload (physical + swap) |

### Monitor Swap Space

```
sar -S 2 3
```
### Advanced Memory Monitoring Commands

Page faults and page ins/outs
```
sar -B 2 3 
```
Huge pages usage 
```
sar -H 2 3 
```
Slabs usage 
```
sar -v 2 3 
```
Swap statistics
```
sar -W 2 3 
```

## Disk I/O Monitoring 

### Monitor Block Device Activity
```
sar -d 2 3
```
**Sample Output** :


| Time | Device | tps | rkB/s | wkB/s | dkB/s | areq-sz | aqu-sz | await | %util |
|------|--------|-----|-------|-------|-------|---------|--------|-------|-------|
| 10:00:01 | sda | 45 | | | | | | | |



