# htop: Comprehensive Guide and Usage

## What is htop?

htop is an interactive process viewer and system monitor for Unix-like systems. It's an advanced alternative to the traditional top command, offering a more user-friendly and feature-rich interface.

Key features:
- Color-coded, real-time view of system processes
- Tree view of processes
- CPU, memory, and swap usage meters
- Ability to scroll vertically and horizontally
- Mouse support for easier navigation and control

## Installation

### On Ubuntu/Debian:
```bash
sudo apt update
sudo apt install htop
```

### On CentOS/RHEL:
```bash
sudo yum install epel-release
sudo yum install htop
```

### On macOS (using Homebrew):
```bash
brew install htop
```

## Basic Usage

To start htop, simply type in your terminal:
```bash
htop
```

## Interface Overview

1. Header: Shows system summary (CPU, memory, swap usage)
2. Process List: Displays running processes
3. Footer: Shows available commands

## Navigation and Control

- Up/Down Arrow Keys: Scroll through process list
- Left/Right Arrow Keys: Scroll horizontally
- PgUp/PgDn: Scroll page by page
- Home/End: Jump to top/bottom of the list
- F1-F10: Access various functions (help, setup, search, etc.)

## Key Commands

- F1: Help
- F2: Setup (customize display)
- F3: Search
- F4: Filter
- F5: Tree view
- F6: SortBy
- F7: Nice (change priority)
- F8: Nice (change priority)
- F9: Kill (send signal)
- F10: Quit

## Advanced Usage

### Customizing the Display

1. Press F2 to enter Setup
2. Navigate using arrow keys
3. Use Space to toggle options
4. Press F10 to save and exit

### Sorting Processes

1. Press F6
2. Select sorting criterion (e.g., CPU%, MEM%, TIME+)
3. Press Enter

### Filtering Processes

1. Press F4
2. Enter filter text
3. Press Enter

### Tree View

Press F5 to toggle between flat list and tree view of processes.

## HPC-Specific Usage

In HPC environments, htop is particularly useful for:

1. Monitoring Node Performance
   - Watch CPU and memory usage across all cores
   - Identify processes consuming excessive resources

2. Job Monitoring
   - Use filtering to focus on specific user's processes
   - Monitor resource usage of running jobs

3. System Health Checks
   - Quick overview of system load and resource utilization
   - Identify potential resource contention issues

4. Debugging
   - Locate and analyze misbehaving processes
   - Check for unexpected system processes that might interfere with job performance

## Best Practices for HPC Use

1. Regular Checks: Periodically run htop on compute nodes to ensure optimal performance.

2. User Education: Teach users to use htop for monitoring their own jobs.

3. Custom Configurations: Create and distribute custom htop configurations tailored for your HPC environment.

4. Integration with Scripts: Use htop's non-interactive mode in scripts for automated monitoring:
   ```bash
   htop -C | head -n 20 > system_status.txt
   ```

5. GPU Monitoring: While htop doesn't show GPU usage directly, use it alongside GPU monitoring tools for a complete picture.

## Troubleshooting Common Issues

1. High Load Average: Check for processes consuming excessive CPU.
2. Memory Leaks: Look for processes with continuously growing memory usage.
3. Zombie Processes: Identify and investigate any zombie processes.
4. I/O Wait: High wa% in CPU meters indicates I/O bottlenecks.

Remember, while htop is powerful, it should be used alongside other monitoring tools for comprehensive system oversight in HPC environments.

# Using htop for Troubleshooting Performance Issues in HPC

## 1. High CPU Usage

### Symptoms in htop:
- High overall CPU usage percentage
- Individual cores showing consistently high utilization

### Troubleshooting Steps:
1. Sort processes by CPU usage (F6 > CPU%)
2. Identify top CPU-consuming processes
3. Check if these align with expected high-performance jobs
4. Look for unexpected processes using significant CPU time

### Potential Issues:
- Runaway processes
- Inefficient job scripts
- Background system tasks interfering with job performance

## 2. Memory Issues

### Symptoms in htop:
- High memory usage bar
- Significant swap usage

### Troubleshooting Steps:
1. Sort processes by memory usage (F6 > MEM%)
2. Check for processes using unexpectedly large amounts of memory
3. Monitor for growing memory usage, indicating potential memory leaks

### Potential Issues:
- Memory leaks in user applications
- Jobs exceeding allocated memory
- Improper memory management in scripts

## 3. I/O Bottlenecks

### Symptoms in htop:
- High 'wa' (I/O wait) percentage in CPU bars
- Processes stuck in D (uninterruptible sleep) state

### Troubleshooting Steps:
1. Identify processes with high I/O wait
2. Use the detailed process view (F2 > "Detailed CPU Time") to see I/O wait per process
3. Check for processes doing excessive disk reads/writes

### Potential Issues:
- Inefficient I/O patterns in applications
- Network file system (NFS) issues
- Disk hardware problems

## 4. Process Priority Issues

### Symptoms in htop:
- Critical processes not getting enough CPU time
- Lower priority jobs interfering with high-priority tasks

### Troubleshooting Steps:
1. Check the 'NI' (nice) column for process priorities
2. Use F7/F8 to adjust priorities if necessary
3. Look for processes with unexpectedly high priorities

### Potential Issues:
- Incorrectly set job priorities
- System processes interfering with job performance

## 5. Zombie Processes

### Symptoms in htop:
- Processes shown in red with 'Z' state

### Troubleshooting Steps:
1. Identify the parent of zombie processes
2. Check if the parent process is functioning correctly

### Potential Issues:
- Improperly terminated jobs
- Bugs in job scripts not cleaning up child processes

## 6. Load Balancing Issues

### Symptoms in htop:
- Uneven CPU usage across cores
- Some cores heavily loaded while others are idle

### Troubleshooting Steps:
1. Use the bar graph view to visualize CPU usage across all cores
2. Check if multi-threaded applications are utilizing cores effectively

### Potential Issues:
- Poor thread allocation in parallel jobs
- Improper CPU affinity settings

## 7. Resource Contention

### Symptoms in htop:
- Multiple high-demand processes competing for resources
- Fluctuating CPU and memory usage

### Troubleshooting Steps:
1. Identify competing processes
2. Check job scheduling logs to understand why competing jobs are on the same node

### Potential Issues:
- Inefficient job scheduling
- Overcommitment of node resources

## Best Practices for Using htop in Troubleshooting

1. Baseline Comparison: Regularly capture htop output during normal operation to have a baseline for comparison.

2. Continuous Monitoring: Use htop in conjunction with logging tools to capture data over time, helping identify intermittent issues.

3. Custom Views: Create custom htop configurations (F2) tailored for specific troubleshooting scenarios.

4. Correlation with Logs: Always correlate htop observations with system and application logs for a complete picture.

5. User Education: Train users to provide htop screenshots or output when reporting performance issues.

6. Integration with Other Tools: Use htop alongside specialized tools (e.g., iostat, vmstat, nvidia-smi) for comprehensive diagnostics.

Remember, while htop is powerful for identifying symptoms, root cause analysis often requires deeper investigation using additional tools and log analysis.