# lspci: Comprehensive Guide and Troubleshooting Usage

## What is lspci?

lspci is a command-line utility for displaying information about PCI buses and devices connected to them. It's an essential tool for system administrators and troubleshooters, providing detailed information about hardware components such as network adapters, graphics cards, and storage controllers.

## Basic Usage

To list all PCI devices:
```bash
lspci
```

This command displays a basic list of all PCI devices with their addresses and descriptions.

## Common Options

1. Verbose output:
   ```bash
   lspci -v
   ```

2. Very verbose output:
   ```bash
   lspci -vv
   ```

3. Kernel drivers in use:
   ```bash
   lspci -k
   ```

4. Numeric IDs:
   ```bash
   lspci -n
   ```

5. Device tree view:
   ```bash
   lspci -t
   ```

## Advanced Usage for Troubleshooting

1. Display PCI configuration space:
   ```bash
   lspci -x
   ```

2. Show specific device details:
   ```bash
   lspci -s 00:02.0 -v
   ```
   Replace `00:02.0` with the address of the device you're interested in.

3. Display IRQ information:
   ```bash
   lspci -vv | grep IRQ
   ```

4. Show power management information:
   ```bash
   lspci -vv | grep -i power
   ```

5. Check for errors:
   ```bash
   lspci -vv | grep -i error
   ```

## Troubleshooting Scenarios

### 1. Missing or Unrecognized Hardware

If a device is not showing up or is not recognized:

a. Check if the device is listed:
   ```bash
   lspci
   ```

b. If listed, check for driver information:
   ```bash
   lspci -k | grep -A 3 "VGA compatible controller"
   ```
   Replace "VGA compatible controller" with the device type you're troubleshooting.

c. If not listed, it could indicate a hardware failure or connection issue.

### 2. Network Issues

For troubleshooting network adapter problems:

a. Identify the network adapter:
   ```bash
   lspci | grep -i ethernet
   ```

b. Check detailed information and driver:
   ```bash
   lspci -vv -s $(lspci | grep -i ethernet | awk '{print $1}')
   ```

### 3. GPU Problems

For graphics card issues:

a. Identify the GPU:
   ```bash
   lspci | grep -i vga
   ```

b. Check detailed information, including current driver:
   ```bash
   lspci -vv -s $(lspci | grep -i vga | awk '{print $1}')
   ```

### 4. Storage Controller Issues

For problems with storage devices:

a. Identify storage controllers:
   ```bash
   lspci | grep -i storage
   ```

b. Check detailed information and driver:
   ```bash
   lspci -vv -s $(lspci | grep -i storage | awk '{print $1}')
   ```

### 5. PCI Express Link Speed Issues

To check if devices are running at the expected PCIe speed:

```bash
lspci -vv | grep -E "LnkCap:|LnkSta:"
```

### 6. IRQ Conflicts

To check for potential IRQ conflicts:

```bash
lspci -vv | grep IRQ
```

## HPC-Specific Troubleshooting

1. InfiniBand/High-Speed Network Adapters:
   ```bash
   lspci | grep -i infiniband
   lspci -vv -s $(lspci | grep -i infiniband | awk '{print $1}')
   ```

2. GPU Cluster Verification:
   ```bash
   lspci | grep -i nvidia
   lspci -vv -s $(lspci | grep -i nvidia | awk '{print $1}')
   ```

3. RAID Controller Checks:
   ```bash
   lspci | grep -i raid
   lspci -vv -s $(lspci | grep -i raid | awk '{print $1}')
   ```

4. NVMe Storage Performance:
   ```bash
   lspci | grep -i nvme
   lspci -vv -s $(lspci | grep -i nvme | awk '{print $1}') | grep 'LnkSta:'
   ```

## Best Practices for Troubleshooting with lspci

1. Baseline Comparisons: Keep a baseline lspci output for each server type in your HPC environment.

2. Regular Audits: Periodically run lspci checks to ensure all expected hardware is present and properly recognized.

3. Driver Verification: Use lspci -k to verify that the correct drivers are loaded for critical components.

4. Performance Checks: Use lspci to verify that high-performance components (like GPUs and network adapters) are running at the expected PCIe link speeds.

5. Error Logging: Incorporate lspci checks into your system logging and monitoring scripts to catch hardware issues early.

6. Consistency Checks: In HPC clusters, use lspci in conjunction with configuration management tools to ensure hardware consistency across nodes.

## Scripting for Automated Checks

Here's a sample script for automated hardware checks:

```bash
#!/bin/bash

echo "Hardware Verification Report"
echo "============================"

# Check for expected number of GPUs
gpu_count=$(lspci | grep -i nvidia | wc -l)
echo "GPU Count: $gpu_count"
if [ $gpu_count -ne 4 ]; then
    echo "WARNING: Expected 4 GPUs, found $gpu_count"
fi

# Check InfiniBand link status
ib_status=$(lspci -vv | grep -A 1 "InfiniBand" | grep "LnkSta:" | awk '{print $2, $3}')
echo "InfiniBand Link Status: $ib_status"
if [[ $ib_status != *"Speed 8GT/s"* ]]; then
    echo "WARNING: InfiniBand not running at expected speed"
fi

# Check for any PCI errors
pci_errors=$(lspci -vv | grep -i error)
if [ ! -z "$pci_errors" ]; then
    echo "PCI Errors Detected:"
    echo "$pci_errors"
fi

echo "============================"
```

Remember, while lspci is a powerful diagnostic tool, it should be used in conjunction with other system utilities and hardware-specific tools for comprehensive troubleshooting in HPC environments.


# Advanced lspci Scripts for Automated Hardware Verification in HPC

## 1. Core Script Structure

Start with a modular script structure for flexibility and maintainability:

```bash
#!/bin/bash

# Configuration
source config.sh

# Function definitions
source functions.sh

# Main execution
main() {
    log_init
    check_gpu
    check_network
    check_storage
    check_pcie_speeds
    generate_report
}

main
```

## 2. Configuration File (config.sh)

Create a separate configuration file for easy customization:

```bash
# config.sh

# Expected hardware configurations
EXPECTED_GPU_COUNT=4
EXPECTED_GPU_MODEL="NVIDIA A100"
EXPECTED_NETWORK="Mellanox ConnectX-6"
EXPECTED_STORAGE="Intel P4610"

# Thresholds
MIN_PCIE_SPEED="16 GT/s"
MAX_TEMP_GPU=85
MAX_TEMP_CPU=80

# Logging
LOG_FILE="/var/log/hw_verify.log"
```

## 3. Function Definitions (functions.sh)

Implement detailed checking functions:

```bash
# functions.sh

log_init() {
    echo "Hardware Verification - $(date)" > $LOG_FILE
}

log_message() {
    echo "[$(date +%Y-%m-%d\ %H:%M:%S)] $1" >> $LOG_FILE
}

check_gpu() {
    log_message "Checking GPUs..."
    gpu_info=$(lspci | grep -i nvidia)
    gpu_count=$(echo "$gpu_info" | wc -l)
    
    if [ $gpu_count -ne $EXPECTED_GPU_COUNT ]; then
        log_message "ERROR: Expected $EXPECTED_GPU_COUNT GPUs, found $gpu_count"
    else
        log_message "GPU count correct: $gpu_count"
    fi

    if ! echo "$gpu_info" | grep -q "$EXPECTED_GPU_MODEL"; then
        log_message "ERROR: GPU model mismatch. Expected $EXPECTED_GPU_MODEL"
    else
        log_message "GPU model correct: $EXPECTED_GPU_MODEL"
    fi

    # Check GPU temperatures
    temperatures=$(nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader)
    for temp in $temperatures; do
        if [ $temp -gt $MAX_TEMP_GPU ]; then
            log_message "WARNING: GPU temperature $temp exceeds maximum $MAX_TEMP_GPU"
        fi
    done
}

check_network() {
    log_message "Checking Network Adapters..."
    network_info=$(lspci | grep -i ethernet)
    
    if ! echo "$network_info" | grep -q "$EXPECTED_NETWORK"; then
        log_message "ERROR: Network adapter mismatch. Expected $EXPECTED_NETWORK"
    else
        log_message "Network adapter correct: $EXPECTED_NETWORK"
    fi

    # Check link status
    for adapter in $(lspci | grep Ethernet | cut -d' ' -f1); do
        link_status=$(lspci -vv -s $adapter | grep 'LnkSta:')
        if ! echo "$link_status" | grep -q "Speed 25GT/s"; then
            log_message "WARNING: Network adapter $adapter not at full speed"
        fi
    done
}

check_storage() {
    log_message "Checking Storage Controllers..."
    storage_info=$(lspci | grep -i storage)
    
    if ! echo "$storage_info" | grep -q "$EXPECTED_STORAGE"; then
        log_message "ERROR: Storage controller mismatch. Expected $EXPECTED_STORAGE"
    else
        log_message "Storage controller correct: $EXPECTED_STORAGE"
    fi

    # Check NVMe speeds
    nvme_drives=$(lspci | grep -i nvme | cut -d' ' -f1)
    for drive in $nvme_drives; do
        speed=$(lspci -vv -s $drive | grep 'LnkSta:' | awk '{print $3}')
        if [[ "$speed" != "$MIN_PCIE_SPEED" ]]; then
            log_message "WARNING: NVMe drive $drive running at $speed, expected $MIN_PCIE_SPEED"
        fi
    done
}

check_pcie_speeds() {
    log_message "Checking PCIe Speeds..."
    pcie_info=$(lspci -vv | grep -E 'LnkCap:|LnkSta:')
    
    while read -r line; do
        if [[ $line == *LnkCap* ]]; then
            cap=$(echo $line | awk '{print $3}')
        elif [[ $line == *LnkSta* ]]; then
            sta=$(echo $line | awk '{print $3}')
            if [[ "$sta" != "$cap" ]]; then
                log_message "WARNING: PCIe link not running at full capacity. Capable: $cap, Status: $sta"
            fi
        fi
    done <<< "$pcie_info"
}

generate_report() {
    log_message "Generating Final Report..."
    error_count=$(grep ERROR $LOG_FILE | wc -l)
    warning_count=$(grep WARNING $LOG_FILE | wc -l)
    
    echo "===============================" >> $LOG_FILE
    echo "Hardware Verification Summary" >> $LOG_FILE
    echo "Errors: $error_count" >> $LOG_FILE
    echo "Warnings: $warning_count" >> $LOG_FILE
    echo "===============================" >> $LOG_FILE

    if [ $error_count -gt 0 ] || [ $warning_count -gt 0 ]; then
        echo "Issues detected. Please review the log file: $LOG_FILE"
        exit 1
    else
        echo "All hardware checks passed successfully."
        exit 0
    fi
}
```

## 4. Advanced Features

### 4.1 Performance Benchmarking

Add functions to run quick performance benchmarks:

```bash
run_gpu_benchmark() {
    log_message "Running GPU Benchmark..."
    # Use a tool like nvidia-smi or a custom CUDA benchmark
    nvidia-smi --query-gpu=utilization.gpu --format=csv,noheader | while read utilization; do
        if [ $utilization -lt 90 ]; then
            log_message "WARNING: GPU utilization below 90% during benchmark"
        fi
    done
}

run_network_benchmark() {
    log_message "Running Network Benchmark..."
    # Use a tool like iperf3
    iperf3 -c $IPERF_SERVER -t 10 | grep receiver | awk '{print $7}' | while read bandwidth; do
        if (( $(echo "$bandwidth < 90" | bc -l) )); then
            log_message "WARNING: Network bandwidth below 90 Gbits/sec"
        fi
    done
}
```

### 4.2 Historical Tracking

Implement a function to compare current results with historical data:

```bash
compare_historical() {
    current_hash=$(lspci | md5sum)
    if [ -f "/var/log/hw_hash.md5" ]; then
        old_hash=$(cat /var/log/hw_hash.md5)
        if [ "$current_hash" != "$old_hash" ]; then
            log_message "WARNING: Hardware configuration changed since last check"
            diff <(lspci) <(lspci_old) | while read line; do
                log_message "Change: $line"
            done
        fi
    fi
    echo $current_hash > /var/log/hw_hash.md5
    lspci > /var/log/lspci_old
}
```

### 4.3 Integration with Cluster Management

For HPC clusters, integrate with your cluster management system:

```bash
check_slurm_config() {
    log_message "Verifying Slurm Configuration..."
    gpu_count=$(lspci | grep -i nvidia | wc -l)
    slurm_gpus=$(scontrol show node $HOSTNAME | grep "Gres=" | grep -oP "gpu:\K\d+")
    
    if [ "$gpu_count" != "$slurm_gpus" ]; then
        log_message "ERROR: Mismatch between physical GPUs ($gpu_count) and Slurm config ($slurm_gpus)"
    fi
}
```

## 5. Error Handling and Reporting

Implement robust error handling and reporting:

```bash
handle_error() {
    local error_message="$1"
    log_message "ERROR: $error_message"
    send_alert "$error_message"
}

send_alert() {
    # Implement your alerting mechanism (e.g., email, Slack notification)
    echo "$1" | mail -s "Hardware Verification Alert" admin@example.com
}

# Wrap main functions with error handling
check_gpu() {
    log_message "Checking GPUs..."
    if ! nvidia-smi &>/dev/null; then
        handle_error "NVIDIA SMI failed. Check GPU drivers."
        return 1
    fi
    # ... rest of the function
}
```

## 6. Scheduling and Automation

Set up a cron job to run the script regularly:

```bash
# Add to /etc/cron.d/hw_verify
0 1 * * * root /path/to/hw_verify.sh
```

## 7. Visualization and Reporting

Generate visual reports using tools like gnuplot:

```bash
generate_visual_report() {
    # Generate data file
    echo "Date GPU_Temp Network_Speed" > /tmp/hw_data.dat
    grep "GPU temperature" $LOG_FILE | awk '{print $1, $5}' >> /tmp/hw_data.dat
    grep "Network adapter" $LOG_FILE | awk '{print $1, $8}' >> /tmp/hw_data.dat

    # Generate plot
    gnuplot <<EOF
    set terminal png
    set output "/var/www/html/hw_report.png"
    set xlabel "Date"
    set ylabel "Value"
    plot "/tmp/hw_data.dat" using 1:2 title "GPU Temp" with lines, \
         "/tmp/hw_data.dat" using 1:3 title "Network Speed" with lines
EOF
}
```

Remember to adapt these scripts to your specific HPC environment, hardware configurations, and operational needs. Regularly review and update the scripts to ensure they remain relevant and effective for your evolving infrastructure.