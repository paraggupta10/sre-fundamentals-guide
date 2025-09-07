# Linux System Monitoring: Top Command & Load Averages

## Overview

System monitoring is a critical skill for SRE and DevOps professionals. The `top` command and understanding load averages are fundamental tools for real-time system performance analysis, process management, and troubleshooting performance issues in production environments.

This guide covers the `top` command interface, load average interpretation, and practical monitoring techniques essential for maintaining system reliability and performance.

## Table of Contents

- [The Top Command](#the-top-command)
  - [Interface Overview](#interface-overview)
  - [Summary Area Details](#summary-area-details)
  - [Process List](#process-list)
  - [Interactive Commands](#interactive-commands)
- [Load Averages Explained](#load-averages-explained)
  - [What is Load Average](#what-is-load-average)
  - [Calculation and Interpretation](#calculation-and-interpretation)
  - [Multi-Core Systems](#multi-core-systems)
- [SRE Relevance](#sre-relevance)
- [Common Monitoring Scenarios](#common-monitoring-scenarios)
- [Performance Optimization](#performance-optimization)
- [Interview Questions & Answers](#interview-questions--answers)
- [Hands-On Exercises](#hands-on-exercises)
- [References](#references)

---

## The Top Command

### Basic Usage

```bash
# Launch top interactively
top

# Show version and variant
top -v

# Sort by specific column
top -o %CPU
top -o %MEM

# Monitor specific user
top -u root
top -u username

# Show threads instead of processes
top -H

# Show full command paths
top -c
```

### Interface Overview

The `top` command displays a dynamic, real-time view of system processes and resource utilization. The interface is divided into two main sections:

1. **Summary Area** (upper half): System statistics and resource usage
2. **Process List** (lower half): Currently running processes with detailed information

### Summary Area Details

#### 1. System Information Line
```
top - 15:39:37 up 90 days, 15:26,  2 users,  load average: 0.15, 0.18, 0.12
```

- **Current Time**: `15:39:37`
- **System Uptime**: `90 days, 15:26` (days, hours:minutes)
- **Active User Sessions**: `2 users` (TTY and PTY sessions)
- **Load Average**: `0.15, 0.18, 0.12` (1, 5, 15-minute averages)

#### 2. Tasks Summary
```
Tasks: 127 total,   1 running, 126 sleeping,   0 stopped,   0 zombie
```

**Process States:**
- **Running (R)**: Currently executing or ready to execute
- **Sleeping (S/D)**: Waiting for events or I/O operations
  - **S**: Interruptible sleep (waiting for events)
  - **D**: Uninterruptible sleep (waiting for I/O)
- **Stopped (T)**: Suspended by job control signals (Ctrl+Z)
- **Zombie (Z)**: Terminated but parent hasn't collected exit status

#### 3. CPU Usage Statistics
```
%Cpu(s):  2.3 us,  1.1 sy,  0.0 ni, 96.5 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
```

**CPU Time Breakdown:**
- **us (User Space)**: Time spent executing user processes
- **sy (System/Kernel)**: Time spent executing kernel processes
- **ni (Nice)**: Time spent on processes with modified nice values
- **id (Idle)**: CPU idle time
- **wa (I/O Wait)**: Time waiting for I/O operations to complete
- **hi (Hardware Interrupts)**: Time handling hardware interrupts
- **si (Software Interrupts)**: Time handling software interrupts
- **st (Steal Time)**: Time stolen by hypervisor (virtualized environments)

#### 4. Memory Usage
```
MiB Mem :   7936.2 total,   1234.5 free,   2345.6 used,   4356.1 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   5123.4 avail Mem
```

**Memory Fields:**
- **Total**: Total physical RAM available
- **Free**: Unused memory
- **Used**: Memory in use by processes
- **Buff/Cache**: Memory used for disk buffers and cache
- **Avail Mem**: Memory available for new processes (without swapping)

**Swap Fields:**
- **Total**: Total swap space configured
- **Free**: Unused swap space
- **Used**: Swap space currently in use

### Process List

#### Column Descriptions

| Column | Description |
|--------|-------------|
| **PID** | Process ID (unique identifier) |
| **USER** | Process owner |
| **PR** | Priority (kernel's view of process priority) |
| **NI** | Nice value (-20 to +19, lower = higher priority) |
| **VIRT** | Virtual memory used by process |
| **RES** | Physical memory (RAM) used by process |
| **SHR** | Shared memory with other processes |
| **S** | Process state (R/S/D/T/Z) |
| **%CPU** | Percentage of CPU time used |
| **%MEM** | Percentage of physical memory used |
| **TIME+** | Total CPU time used since process started |
| **COMMAND** | Process name or command line |

#### Memory Fields Explained

- **VIRT (Virtual Memory)**: Total virtual memory used by process
  - Includes: program code, data, shared libraries, swapped memory
  - Can be larger than physical RAM
  
- **RES (Resident Memory)**: Physical RAM currently used
  - Actually loaded in memory (not swapped)
  - Most important for memory usage analysis
  
- **SHR (Shared Memory)**: Memory shared with other processes
  - Libraries, shared memory segments
  - Counted in multiple processes' RES values

### Interactive Commands

#### Navigation and Display
```bash
# Navigation
↑/↓ arrows    # Scroll through process list
Page Up/Down  # Scroll by pages
Home/End      # Jump to top/bottom

# Display toggles
c             # Toggle command line/name display
f             # Add/remove columns (field management)
H             # Toggle threads view
V             # Forest view (process tree)
```

#### Sorting
```bash
P             # Sort by CPU usage (default)
M             # Sort by memory usage
N             # Sort by process ID
T             # Sort by running time
R             # Reverse sort order
```

#### Process Management
```bash
k             # Kill process (prompts for PID and signal)
r             # Renice process (change priority)
u             # Filter by user
o/O           # Add filter expressions
=             # Clear all filters
```

#### Display Customization
```bash
t             # Toggle CPU summary display style
m             # Toggle memory summary display style
1             # Toggle individual CPU core display
W             # Save current configuration to ~/.toprc
```

---

## Load Averages Explained

### What is Load Average

**Load Average** represents the average system load over a period of time. It measures the number of processes that are either:
- Currently running on the CPU
- Waiting in the run queue to be executed
- Waiting for I/O operations to complete (in uninterruptible sleep state D)

### The Three Numbers

```bash
load average: 0.15, 0.18, 0.12
```

- **First number (0.15)**: 1-minute load average
- **Second number (0.18)**: 5-minute load average  
- **Third number (0.12)**: 15-minute load average

### Calculation and Interpretation

#### Single-Core System
- **0.00**: System completely idle
- **0.50**: System at 50% capacity
- **1.00**: System at 100% capacity (fully utilized)
- **1.50**: System overloaded by 50% (150% demand)
- **2.00**: System overloaded by 100% (200% demand)

#### Multi-Core Systems
For multi-core systems, divide load average by number of CPU cores:

```bash
# Check number of CPU cores
nproc
grep -c ^processor /proc/cpuinfo

# Example: 4-core system with load average 2.0
# Actual utilization: 2.0 ÷ 4 = 0.5 (50% capacity)
```

#### Load Average Thresholds

| Load/Core Ratio | System State | Action Required |
|-----------------|--------------|-----------------|
| 0.00 - 0.70 | Optimal | None |
| 0.70 - 1.00 | Acceptable | Monitor |
| 1.00 - 1.50 | Concerning | Investigate |
| 1.50+ | Critical | Immediate action |

### Exponential Moving Average

Load averages use exponential moving average, not simple arithmetic average:
- Recent samples have more weight than older samples
- Provides smoother representation of system load trends
- 1-minute average responds quickly to load changes
- 15-minute average shows long-term trends

---

## SRE Relevance

### Production Monitoring

#### Key Metrics to Watch
1. **Load Average Trends**: Identify performance degradation
2. **Memory Utilization**: Prevent OOM (Out of Memory) conditions
3. **Process States**: Detect hung processes or I/O bottlenecks
4. **CPU Wait Time**: Identify storage performance issues

#### Alerting Thresholds
```bash
# Example alerting rules
Load Average (1-min) > 0.8 * CPU_CORES  # Warning
Load Average (1-min) > 1.2 * CPU_CORES  # Critical
Memory Usage > 85%                       # Warning
Memory Usage > 95%                       # Critical
```

### Capacity Planning
- **Baseline Performance**: Establish normal operating ranges
- **Growth Trends**: Plan infrastructure scaling
- **Resource Allocation**: Optimize container/VM resources

### Incident Response
- **Quick System Assessment**: Rapid health check during incidents
- **Root Cause Analysis**: Identify resource bottlenecks
- **Performance Impact**: Measure incident effects on system performance

---

## Common Monitoring Scenarios

### High Load Average Investigation

#### Step 1: Check Current Load
```bash
# Quick load check
uptime
cat /proc/loadavg

# Detailed analysis
top
htop  # If available, more user-friendly
```

#### Step 2: Identify Load Source
```bash
# CPU-bound processes
top -o %CPU

# I/O-bound processes (high D state count)
ps aux | awk '$8 ~ /D/ { print $0 }'

# Check I/O statistics
iostat -x 1 5
iotop  # If available
```

#### Step 3: System Resource Analysis
```bash
# Memory pressure
free -h
cat /proc/meminfo

# Disk I/O
iostat -x 1
vmstat 1

# Network I/O
netstat -i
ss -tuln
```

### Memory Usage Analysis

#### Memory Leak Detection
```bash
# Monitor memory usage over time
top -p PID -d 1

# Check process memory maps
pmap PID
cat /proc/PID/smaps
```

#### System Memory Pressure
```bash
# Check for OOM killer activity
dmesg | grep -i "killed process"
journalctl -k | grep -i "killed process"

# Monitor swap usage
swapon --show
cat /proc/swaps
```

### Process Management

#### Finding Resource-Heavy Processes
```bash
# Top CPU consumers
ps aux --sort=-%cpu | head -10

# Top memory consumers  
ps aux --sort=-%mem | head -10

# Processes in specific states
ps aux | awk '$8 ~ /D|Z|T/ { print $0 }'
```

#### Process Tree Analysis
```bash
# Process hierarchy
pstree -p
ps auxf

# Parent-child relationships
ps -eo pid,ppid,cmd --forest
```

---

## Performance Optimization

### CPU Optimization

#### Process Priority Management
```bash
# Change process priority (nice value)
# Lower nice = higher priority (-20 to +19)
nice -n 10 command          # Start with lower priority
renice -n 5 -p PID          # Change running process priority

# Real-time priority (use carefully)
chrt -f 99 command          # FIFO scheduling
chrt -r 50 command          # Round-robin scheduling
```

#### CPU Affinity
```bash
# Bind process to specific CPU cores
taskset -c 0,1 command      # Run on cores 0 and 1
taskset -cp 0,1 PID         # Change running process affinity

# Check current affinity
taskset -cp PID
```

### Memory Optimization

#### Memory Usage Reduction
```bash
# Clear system caches (use carefully in production)
sync
echo 1 > /proc/sys/vm/drop_caches  # Page cache
echo 2 > /proc/sys/vm/drop_caches  # Dentries and inodes
echo 3 > /proc/sys/vm/drop_caches  # All caches

# Adjust swappiness (0-100, lower = less swapping)
echo 10 > /proc/sys/vm/swappiness
```

#### Memory Monitoring Tools
```bash
# Detailed memory analysis
cat /proc/meminfo
vmstat -s

# Per-process memory usage
smem -p          # If available
ps_mem           # If available
```

### I/O Optimization

#### Disk I/O Analysis
```bash
# I/O statistics
iostat -x 1 5

# Per-process I/O
iotop -o         # Show only active I/O processes
pidstat -d 1     # Per-process disk statistics
```

#### I/O Scheduling
```bash
# Check current I/O scheduler
cat /sys/block/sda/queue/scheduler

# Change I/O scheduler
echo deadline > /sys/block/sda/queue/scheduler
echo cfq > /sys/block/sda/queue/scheduler
echo noop > /sys/block/sda/queue/scheduler
```

---

## Interview Questions & Answers

### Common Questions

#### Q1: What does a load average of 1.5 mean on a 4-core system?
**Answer**: On a 4-core system, a load average of 1.5 means the system is running at 37.5% capacity (1.5 ÷ 4 = 0.375). This indicates the system is well within normal operating parameters with plenty of available capacity.

#### Q2: What's the difference between %CPU and load average?
**Answer**: 
- **%CPU** shows the percentage of CPU time a specific process used during the last update interval
- **Load Average** shows the average number of processes waiting for CPU time or I/O completion over 1, 5, and 15-minute periods
- %CPU is instantaneous per-process, while load average is a system-wide trend indicator

#### Q3: What does high load average with low CPU utilization indicate?
**Answer**: This typically indicates I/O bottlenecks. Processes are waiting for disk or network I/O operations to complete (in D state), contributing to load average but not consuming CPU cycles. Check with `iostat`, `iotop`, or look for processes in D state.

#### Q4: How do you kill a process that doesn't respond to SIGTERM?
**Answer**:
```bash
# First try graceful termination
kill PID
kill -TERM PID

# If unresponsive, force kill
kill -KILL PID
kill -9 PID

# In top command
# Press 'k', enter PID, then enter signal (9 for SIGKILL)
```

#### Q5: What's the difference between VIRT, RES, and SHR memory in top?
**Answer**:
- **VIRT**: Total virtual memory used (includes swapped memory, shared libraries)
- **RES**: Physical RAM currently used by the process (resident memory)
- **SHR**: Memory shared with other processes (libraries, shared memory segments)
- **RES** is most important for actual memory consumption analysis

### Advanced Questions

#### Q6: How would you troubleshoot a system with consistently high load but you can't find the culprit process?
**Answer**:
1. **Check for short-lived processes**: Use `atop` or `sysstat` to capture historical data
2. **Monitor process creation**: `execsnoop` (if available) or audit logs
3. **Check kernel threads**: Look for kernel processes in brackets `[kworker]`
4. **I/O analysis**: Use `iotop`, `iostat` to identify I/O-bound processes
5. **System calls**: Use `strace` on suspicious processes
6. **Check for containers**: Docker/Kubernetes processes might be hidden

#### Q7: Explain the relationship between load average and system performance.
**Answer**:
Load average measures system saturation, not utilization:
- **Below 1.0 per core**: System responsive, resources available
- **At 1.0 per core**: System fully utilized but not overloaded
- **Above 1.0 per core**: System overloaded, requests queuing
- **Trend analysis**: Compare 1, 5, 15-minute averages to understand load direction
- **Context matters**: I/O-heavy workloads may show high load with low CPU usage

#### Q8: How do you optimize a system showing high memory usage in top?
**Answer**:
1. **Identify memory hogs**: Sort by %MEM, check VIRT vs RES
2. **Check for memory leaks**: Monitor process memory over time
3. **Analyze cache usage**: High buff/cache is usually normal
4. **Swap analysis**: Check if system is swapping heavily
5. **Tune applications**: Adjust JVM heap, database buffers, etc.
6. **System tuning**: Adjust `vm.swappiness`, enable/configure zswap
7. **Add memory**: If consistently high, consider hardware upgrade

#### Q9: What's the significance of different process states in production monitoring?
**Answer**:
- **R (Running)**: Normal, indicates active processing
- **S (Sleeping)**: Normal, waiting for events/timers
- **D (Uninterruptible Sleep)**: Concerning if many processes, indicates I/O bottlenecks
- **T (Stopped)**: Unusual in production, may indicate debugging or job control
- **Z (Zombie)**: Problematic if accumulating, indicates parent process issues
- **Monitor D and Z states**: High counts suggest system problems

#### Q10: How would you use top for capacity planning?
**Answer**:
1. **Baseline establishment**: Monitor normal operating patterns
2. **Peak usage identification**: Track maximum resource utilization
3. **Growth trend analysis**: Compare load patterns over time
4. **Resource headroom**: Ensure sufficient capacity for spikes
5. **Bottleneck identification**: Find limiting resources (CPU, memory, I/O)
6. **Scaling triggers**: Define thresholds for horizontal/vertical scaling
7. **Historical data**: Combine with monitoring tools for long-term trends

---

## Hands-On Exercises

### Exercise 1: Basic Top Navigation
```bash
# Launch top and practice navigation
top

# Try these commands:
# 1. Press 'M' to sort by memory
# 2. Press 'P' to sort by CPU
# 3. Press 'c' to toggle command display
# 4. Press 'H' to show threads
# 5. Press 'u' and enter your username
# 6. Press 'q' to quit
```

### Exercise 2: Load Average Analysis
```bash
# Generate CPU load
yes > /dev/null &
yes > /dev/null &

# Monitor load in another terminal
watch -n 1 'uptime; echo; top -bn1 | head -5'

# Clean up
killall yes
```

### Exercise 3: Memory Monitoring
```bash
# Create memory pressure (be careful with size)
stress --vm 1 --vm-bytes 512M --timeout 60s &

# Monitor in top
top -p $(pgrep stress)

# Watch memory usage
watch -n 1 'free -h; echo; cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable"'
```

### Exercise 4: Process Management
```bash
# Start a background process
sleep 300 &
SLEEP_PID=$!

# Practice process management in top
top -p $SLEEP_PID

# Try:
# 1. Press 'r' to renice the process
# 2. Press 'k' to kill the process
# 3. Use different signals (15, 9)
```

### Exercise 5: I/O Monitoring
```bash
# Generate I/O load
dd if=/dev/zero of=/tmp/testfile bs=1M count=1000 &

# Monitor I/O impact
top
# Look for processes in D state
# Check wa (I/O wait) percentage

# Clean up
rm /tmp/testfile
```

### Exercise 6: Custom Top Configuration
```bash
# Customize top display
top

# Configure:
# 1. Press 'f' to add/remove fields
# 2. Add PPID, SWAP, nTH fields
# 3. Press 't' multiple times for CPU display styles
# 4. Press 'm' multiple times for memory display styles
# 5. Press 'W' to save configuration
```

### Exercise 7: Filtering and Searching
```bash
# Practice filtering
top

# Try these filters:
# 1. Press 'o' and enter: COMMAND=ssh
# 2. Press 'o' and enter: %CPU>1.0
# 3. Press 'o' and enter: %MEM>5.0
# 4. Press '=' to clear filters
```

---

## References

### Official Documentation
- [Linux man page for top](https://man7.org/linux/man-pages/man1/top.1.html)
- [proc(5) - /proc filesystem](https://man7.org/linux/man-pages/man5/proc.5.html)
- [Linux Load Averages: Solving the Mystery](http://www.brendangregg.com/blog/2017-08-08/linux-load-averages.html)

### External Resources
- [Boolean World: Guide to Linux Top Command](https://www.booleanworld.com/guide-linux-top-command/)
- [Understanding Linux Load Averages and Monitor Performance](https://www.tecmint.com/understand-linux-load-averages-and-monitor-performance/)

### Related Tools
- `htop` - Enhanced version of top with better interface
- `atop` - Advanced system monitor with historical data
- `iotop` - I/O monitoring by process
- `vmstat` - Virtual memory statistics
- `iostat` - I/O statistics
- `pidstat` - Per-process statistics

### Best Practices
- Monitor load averages in context of system capacity
- Use multiple tools for comprehensive analysis
- Establish baselines for normal system behavior
- Set up automated alerting for critical thresholds
- Regular performance reviews for capacity planning

---

*This guide provides comprehensive coverage of Linux system monitoring fundamentals essential for SRE and DevOps professionals. Master these concepts to effectively monitor, troubleshoot, and optimize system performance in production environments.*
