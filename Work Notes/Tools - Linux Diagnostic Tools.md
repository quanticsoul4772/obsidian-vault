# Linux Diagnostic Tools Guide

## Overview
This guide covers essential diagnostic tools for Linux systems, with specific installation instructions for both Ubuntu and Rocky Linux distributions. These tools are particularly useful for system administrators managing high-performance computing environments.

## Installation Prerequisites

### Ubuntu
```bash
# Update package list
sudo apt update
sudo apt upgrade
```

### Rocky Linux
```bash
# Enable EPEL repository
sudo dnf install epel-release
sudo dnf update

# For NVIDIA tools (if needed)
sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo
sudo dnf clean all
sudo dnf module install nvidia-driver:latest-dkms
```

## Diagnostic Tools

1. Memory Testing
   ```bash
   # Ubuntu
   sudo apt install memtest86+
   
   # Rocky Linux
   sudo dnf install memtest86+
   
   # Usage (both)
   # Reboot and select memtest86+ from GRUB menu
   ```

2. System Stress Testing
   ```bash
   # Ubuntu
   sudo apt install stress-ng
   
   # Rocky Linux
   sudo dnf install stress-ng
   
   # Usage (both)
   stress-ng --cpu 8 --io 4 --vm 2 --vm-bytes 128M --timeout 10s
   ```

3. GPU Monitoring (NVIDIA)
   ```bash
   # Included with NVIDIA drivers
   nvidia-smi
   ```

4. Process Monitoring
   ```bash
   # Ubuntu
   sudo apt install htop
   
   # Rocky Linux
   sudo dnf install htop
   
   # Usage
   htop
   ```

5. Network Performance
   ```bash
   # Ubuntu
   sudo apt install iperf3
   
   # Rocky Linux
   sudo dnf install iperf3
   
   # Usage
   iperf3 -s    # Server mode
   iperf3 -c server_ip    # Client mode
   ```

6. Hardware Monitoring
   ```bash
   # Ubuntu
   sudo apt install lm-sensors
   
   # Rocky Linux
   sudo dnf install lm_sensors
   
   # Usage
   sudo sensors-detect
   sensors
   ```

7. Storage Health
   ```bash
   # Ubuntu
   sudo apt install smartmontools
   
   # Rocky Linux
   sudo dnf install smartmontools
   
   # Usage
   sudo smartctl -a /dev/sda
   ```

8. IPMI Management
   ```bash
   # Ubuntu
   sudo apt install ipmitool
   
   # Rocky Linux
   sudo dnf install ipmitool
   
   # Usage
   ipmitool sensor list
   ```

9. RAID Management
   ```bash
   # Ubuntu
   sudo apt install mdadm
   
   # Rocky Linux
   sudo dnf install mdadm
   
   # Usage
   sudo mdadm --detail /dev/md0
   ```

10. System Statistics
    ```bash
    # Ubuntu
    sudo apt install sysstat dstat nmon
    
    # Rocky Linux
    sudo dnf install sysstat dstat nmon
    
    # Usage
    dstat
    sar -u 1 10
    nmon
    ```

11. Disk Performance
    ```bash
    # Ubuntu
    sudo apt install hdparm fio
    
    # Rocky Linux
    sudo dnf install hdparm fio
    
    # Usage
    sudo hdparm -tT /dev/sda
    fio --name=random-write --ioengine=posixaio --rw=randwrite --bs=4k --size=4g --numjobs=1 --runtime=60
    ```

12. Performance Analysis
    ```bash
    # Ubuntu
    sudo apt install linux-tools-generic
    
    # Rocky Linux
    sudo dnf install perf
    
    # Usage
    sudo perf stat -d program_name
    ```

13. NUMA Control (Rocky Linux)
    ```bash
    sudo dnf install numactl
    numactl --hardware
    ```

14. Hardware Locality
    ```bash
    # Ubuntu
    sudo apt install hwloc
    
    # Rocky Linux
    sudo dnf install hwloc
    
    # Usage
    lstopo
    ```

## Best Practices
1. System Monitoring
   - Regular health checks
   - Performance baselines
   - Alert setup
   - Log rotation

2. Performance Testing
   - Off-peak testing
   - Incremental load
   - Resource monitoring
   - Result documentation

3. Security
   - Proper permissions
   - Access control
   - Audit logging
   - Update management

4. Documentation
   - Configuration changes
   - Test results
   - Performance metrics
   - Troubleshooting steps

## Tags
#diagnostics #tools #monitoring #system-administration #ubuntu #rocky-linux