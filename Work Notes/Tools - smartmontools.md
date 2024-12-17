# smartmontools

## Overview
smartmontools is a package containing utilities (smartctl and smartd) that control and monitor storage systems using the Self-Monitoring, Analysis and Reporting Technology System (SMART). It works with most modern ATA/SATA, SCSI/SAS and NVMe storage devices, enabling early warning of disk degradation and failure through its two main components:
- smartctl: Command-line tool for querying and controlling SMART features
- smartd: Daemon for continuous monitoring and reporting

## Installation
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install smartmontools

# CentOS/RHEL
sudo yum install smartmontools
```

## Basic Usage
```bash
# Check if SMART is available and enabled
smartctl -i /dev/sda

# View SMART health summary
smartctl -H /dev/sda

# Run self-tests
smartctl -t short /dev/sda
smartctl -t long /dev/sda

# View detailed SMART attributes
smartctl -a /dev/sda

# View error logs
smartctl -l error /dev/sda

# Check NVMe drive
smartctl -a /dev/nvme0

# Enable SMART on a drive
smartctl -s on /dev/sda

# View test results
smartctl -l selftest /dev/sda

# Check bad sectors
smartctl -l xerror /dev/sda

# Monitor specific attributes
smartctl -A /dev/sda | grep Temperature_Celsius
```

## Continuous Monitoring with smartd
1. Edit configuration:
```bash
sudo nano /etc/smartd.conf
```

2. Add monitoring directives:
```
/dev/sda -a -o on -S on -s (S/../.././02|L/../../6/03) -m root -M exec /usr/share/smartmontools/smartd-runner
```
This monitors /dev/sda, runs short tests weekly and long tests monthly.

3. Start and enable service:
```bash
sudo systemctl start smartd
sudo systemctl enable smartd
```

## Automation Scripts
```bash
#!/bin/bash
# Disk Health Monitoring Script

# Configuration
LOG_DIR="/var/log/smart-monitor"
ALERT_EMAIL="admin@example.com"
TEMPERATURE_THRESHOLD=50  # Celsius
REALLOCATED_THRESHOLD=10  # Number of reallocated sectors
CHECK_INTERVAL=3600       # Seconds

# Create log directory
mkdir -p "$LOG_DIR"

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_DIR/smart-monitor.log"
}

# Function to get drive list
get_drives() {
    lsblk -d -n -o NAME | grep -E '^sd|^nvme' | sort
}

# Function to check drive health
check_drive_health() {
    local drive="$1"
    local report_file="$LOG_DIR/${drive}_health_$(date +%Y%m%d_%H%M%S).json"
    
    # Get SMART data
    local smart_data=$(smartctl -a "/dev/$drive")
    local health_status=$(echo "$smart_data" | grep -i "SMART overall-health" | awk '{print $NF}')
    local temperature=$(echo "$smart_data" | grep -i "Temperature_Celsius" | awk '{print $10}')
    local reallocated=$(echo "$smart_data" | grep -i "Reallocated_Sector" | awk '{print $10}')
    
    # Generate JSON report
    {
        echo "{"
        echo "  \"timestamp\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\","
        echo "  \"drive\": \"$drive\","
        echo "  \"health_status\": \"$health_status\","
        echo "  \"smart_enabled\": $(smartctl -i "/dev/$drive" | grep -q "SMART support is: Enabled" && echo "true" || echo "false"),"
        echo "  \"temperature_celsius\": $temperature,"
        echo "  \"reallocated_sectors\": $reallocated,"
        echo "  \"power_on_hours\": $(echo "$smart_data" | grep "Power_On_Hours" | awk '{print $10}'),"
        echo "  \"last_test_result\": \"$(smartctl -l selftest "/dev/$drive" | grep "# 1" | awk '{print $NF}')\""
        echo "}"
    } > "$report_file"
    
    # Check for issues
    if [ "$health_status" != "PASSED" ]; then
        log_message "WARNING: Drive $drive health check failed"
        echo "SMART Health Check Failed for drive $drive on $(hostname)" | mail -s "Drive Health Alert" "$ALERT_EMAIL"
    fi
    
    if [ -n "$temperature" ] && [ "$temperature" -gt "$TEMPERATURE_THRESHOLD" ]; then
        log_message "WARNING: Drive $drive temperature above threshold: ${temperature}°C"
        echo "High drive temperature detected on $drive: ${temperature}°C" | mail -s "Drive Temperature Alert" "$ALERT_EMAIL"
    fi
    
    if [ -n "$reallocated" ] && [ "$reallocated" -gt "$REALLOCATED_THRESHOLD" ]; then
        log_message "WARNING: Drive $drive has high reallocated sectors: $reallocated"
        echo "High reallocated sectors on drive $drive: $reallocated" | mail -s "Drive Sector Alert" "$ALERT_EMAIL"
    fi
}

# Main execution
log_message "Starting SMART monitoring"
for drive in $(get_drives); do
    check_drive_health "$drive"
done
```

## HPC-Specific Usage
1. Predictive Failure Analysis:
```bash
#!/bin/bash
for drive in /dev/sd?; do
    smart_status=$(sudo smartctl -H $drive | grep "SMART overall-health")
    if [[ $smart_status != *"PASSED"* ]]; then
        echo "Warning: $drive may be failing!" | mail -s "Disk Failure Warning" admin@example.com
    fi
done
```

2. RAID Array Monitoring:
```bash
#!/bin/bash
for drive in /dev/sd?; do
    echo "Checking $drive:"
    sudo smartctl -H $drive
done
```

3. Performance and Health Metrics:
```bash
# Performance monitoring
smartctl -A /dev/sda | grep -E "Raw_Read_Error_Rate|Seek_Error_Rate|Throughput_Performance"

# Temperature monitoring
smartctl -A /dev/sda | grep Temperature_Celsius

# SSD wear leveling
smartctl -A /dev/sda | grep "Wear_Leveling_Count"
```

## Integration Examples
### Prometheus Integration
```yaml
# prometheus.yml configuration for SMART monitoring
scrape_configs:
  - job_name: 'smartmon'
    static_configs:
      - targets: ['localhost:9100']
    metrics_path: '/metrics'
    params:
      collect[]:
        - smartmon

# Alert rules for disk monitoring
groups:
  - name: disk_alerts
    rules:
      - alert: DiskHealthFailed
        expr: smartmon_health_status == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk health check failed"
          description: "SMART health check failed for disk {{ $labels.device }}"
      
      - alert: DiskTemperatureHigh
        expr: smartmon_temperature_celsius > 50
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "High disk temperature"
          description: "Disk temperature is {{ $value }}°C for {{ $labels.device }}"
```

## Interpreting SMART Data
1. Critical Attributes:
   - Reallocated Sectors Count: Bad sectors that were remapped
   - Current Pending Sectors: Sectors waiting to be remapped
   - Offline Uncorrectable: Sectors that couldn't be read
   - Spin Retry Count: Failed spin-up attempts
   - SSD-specific: Media Wearout Indicator

2. Performance Indicators:
   - Seek Error Rate: Failed seek operations
   - Throughput Performance: Overall performance rating
   - Power-On Hours: Drive lifetime usage
   - Temperature: Operating temperature

## Troubleshooting Guide
1. SMART Not Available
```bash
# Check support
smartctl -i /dev/sda

# Enable SMART
smartctl -s on /dev/sda

# Verify capability
hdparm -I /dev/sda | grep SMART
```

2. Drive Errors
```bash
# Check error logs
smartctl -l error /dev/sda

# View self-test logs
smartctl -l selftest /dev/sda

# Check bad sectors
smartctl -l xerror /dev/sda
```

## Best Practices
1. Monitoring Strategy
   - Regular health checks
   - Temperature monitoring
   - Error log review
   - Performance tracking
   - Proactive testing
   - Automated reporting
   - Trend analysis

2. HPC Environment
   - Schedule tests during low-usage periods
   - Monitor drive temperatures in dense setups
   - Correlate with application I/O metrics
   - Plan preemptive replacements
   - Maintain performance baselines

3. Maintenance
   - Regular firmware updates
   - Scheduled testing
   - Error rate tracking
   - Drive history documentation
   - Spare drive inventory

4. Security and Recovery
   - Backup critical data
   - Document procedures
   - Test recovery processes
   - Maintain spare drives
   - Access control policies

## Tags
#storage #monitoring #hardware #smart #disk-management #hpc