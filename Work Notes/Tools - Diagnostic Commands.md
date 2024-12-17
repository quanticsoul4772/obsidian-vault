# Diagnostic Commands Reference

## System Information Commands

### lshw (Hardware Lister)
[Previous content remains the same...]

### journalctl (System Journal)
**Purpose**: Query and display systemd journal logs
**Command Format**: `journalctl [options]`
**Key Options**:
- `-o json`: Output in JSON format
- `-u unit`: Show logs for specific service
- `-p err..alert`: Show high-priority messages
- `-b`: Logs from current boot
- `-k`: Show kernel messages
- `-f`: Follow new entries

**Sample Script for Automation**:
```bash
#!/bin/bash
# Get recent errors in JSON format
journalctl -p err..alert -o json --since "1 hour ago" > recent_errors.json
# Get boot-related messages
journalctl -b -o json > boot_messages.json
```

**Key Fields to Monitor**:
- PRIORITY
- _SYSTEMD_UNIT
- MESSAGE
- _HOSTNAME
- _BOOT_ID

### htop/top (Process Monitoring)
**Purpose**: Real-time system process monitoring
**Command Format**: `htop -C | head -n 20 > snapshot.txt`
**Batch Mode Options**:
```bash
# Get process snapshot in CSV format
ps -eo pid,ppid,user,%cpu,%mem,state,command --sort=-%cpu | head -n 20 > top_processes.csv
```

**Key Metrics**:
- CPU usage per core
- Memory usage
- Swap usage
- Process tree
- Load average

### strace (System Call Tracer)
**Purpose**: Trace system calls and signals
**Command Format**: `strace [options] command`
**Key Options**:
- `-c`: Summary of system calls
- `-f`: Follow child processes
- `-e trace=network,ipc`: Trace specific calls
- `-o output.txt`: Write to file

**Sample Script for Automation**:
```bash
#!/bin/bash
# Trace specific application
strace -c -f -o app_trace.txt ./application
# Monitor specific system calls
strace -e trace=network,ipc -o network_calls.txt application
```

### sar (System Activity Reporter)
**Purpose**: Collect and report system activity
**Command Format**: `sar [options] [interval] [count]`
**Key Options**:
- `-u`: CPU utilization
- `-r`: Memory utilization
- `-b`: I/O and transfer rate
- `-n DEV`: Network statistics
- `-F`: Filesystem statistics

**Sample Script for Automation**:
```bash
#!/bin/bash
# Collect comprehensive system metrics
sar -u -r -b -n DEV -o system_metrics.sar 1 60
# Convert to CSV format
sadf -d system_metrics.sar -- -u -r -b -n DEV > system_metrics.csv
```

### lsof (List Open Files)
**Purpose**: List open files and processes
**Command Format**: `lsof [options]`
**Key Options**:
- `-i`: Internet connections
- `-u user`: Files by user
- `-p pid`: Files by process ID
- `-F`: Machine-parsable output

**Sample Script for Automation**:
```bash
#!/bin/bash
# Get network connections
lsof -i -F > network_connections.txt
# Get process file handles
lsof -p $(pgrep application) -F > application_files.txt
```

### sysstat Tools
**Purpose**: System performance tools collection
**Components**:
- iostat: I/O statistics
- mpstat: CPU statistics
- pidstat: Process statistics
- tapestat: Tape drive statistics

**Sample Script for Automation**:
```bash
#!/bin/bash
# Collect comprehensive performance data
INTERVAL=5
COUNT=10

iostat -xz $INTERVAL $COUNT > iostat.log &
mpstat -P ALL $INTERVAL $COUNT > mpstat.log &
pidstat -l $INTERVAL $COUNT > pidstat.log &
```

### blktrace (Block Layer I/O Tracing)
**Purpose**: Block layer I/O tracing
**Command Format**: `blktrace [options] device`
**Key Options**:
- `-d device`: Target device
- `-o output`: Output file prefix
- `-w timeout`: Stop after timeout

**Sample Script for Automation**:
```bash
#!/bin/bash
# Trace disk I/O for 60 seconds
blktrace -d /dev/sda -o disk_trace -w 60
# Convert to human-readable format
blkparse disk_trace > disk_trace.txt
```

### perf (Performance Analysis)
**Purpose**: Performance analysis tools
**Command Format**: `perf [command] [options]`
**Key Commands**:
- `stat`: Performance statistics
- `record`: Record system events
- `report`: Display recorded data
- `top`: System profiling

**Sample Script for Automation**:
```bash
#!/bin/bash
# Record system performance for 60 seconds
perf record -a -g -F 99 sleep 60
# Generate report
perf report --stdio > performance_report.txt
```

### systemd-analyze
**Purpose**: Analyze system boot-up performance
**Command Format**: `systemd-analyze [command]`
**Key Commands**:
- `time`: Show boot time
- `blame`: Show unit initialization time
- `critical-chain`: Show critical chain
- `plot`: Generate SVG visualization

**Sample Script for Automation**:
```bash
#!/bin/bash
# Collect boot analysis
systemd-analyze time > boot_time.txt
systemd-analyze blame > service_init_time.txt
systemd-analyze critical-chain > boot_chain.txt
```

## Integration Notes

### Advanced Data Collection Strategy
1. System Health Check:
```bash
#!/bin/bash
# Get detailed CPU info
lscpu -J > cpu_info.json
# Get memory info
free -h --json > memory_info.json
# Get disk info
lsblk -J > block_devices.json
# Get PCI devices
lspci -vvv -nn > pci_devices.txt
```

2. Performance Analysis:
```bash
#!/bin/bash
# CPU performance
mpstat -P ALL 1 5 > cpu_performance.log
# Memory usage
vmstat 1 5 > memory_usage.log
# I/O stats
iostat -xz 1 5 > io_stats.log
```

3. Network Analysis:
```bash
#!/bin/bash
# Network interfaces
ip -j addr > network_interfaces.json
# Network statistics
ss -tunapl > network_connections.txt
# DNS resolution
resolvectl status > dns_status.txt
```

### Error Pattern Detection
Create pattern files for common issues:
```bash
# GPU errors
nvidia-smi -q > gpu_status.txt
grep -E "Error|Warning" gpu_status.txt

# System errors
journalctl -p err..alert --no-pager | grep -E "error|failed|failure" > system_errors.txt

# Hardware errors
mcelog --client | grep -E "Hardware event" > hardware_errors.txt
```

## Tags
#tools #diagnostics #automation #system-admin #performance-monitoring