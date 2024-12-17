# RAID Management with mdadm

## Overview
mdadm is the Linux utility for managing software RAID arrays. It supports creating, assembling, monitoring, and maintaining MD (multiple devices) RAID configurations.

## Installation
```bash
# Install mdadm
sudo apt install mdadm    # Ubuntu/Debian
sudo dnf install mdadm    # RHEL/Rocky
```

## Essential Commands

### RAID Information
```bash
# Show RAID details
mdadm --detail /dev/md0

# Scan for RAID arrays
mdadm --scan

# Check array status
cat /proc/mdstat

# Check array consistency
echo check > /sys/block/md0/md/sync_action
```

### RAID Creation
```bash
# Create RAID 1 (Mirror)
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1

# Create RAID 5
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb1 /dev/sdc1 /dev/sdd1

# Create RAID 10
mdadm --create /dev/md0 --level=10 --raid-devices=4 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
```

### RAID Management
```bash
# Add a spare disk
mdadm /dev/md0 --add /dev/sdd1

# Remove a disk
mdadm /dev/md0 --remove /dev/sdd1

# Mark disk as faulty
mdadm /dev/md0 --fail /dev/sdd1

# Stop array
mdadm --stop /dev/md0

# Assemble array
mdadm --assemble /dev/md0
```

## Automation Scripts

### RAID Health Check
```bash
#!/bin/bash
# Check RAID health and send alerts

check_raid_health() {
    local report_file="raid_health_$(date +%Y%m%d_%H%M%S).log"
    
    echo "RAID Health Check - $(date)" > "$report_file"
    
    # Check all arrays
    for md in /dev/md*; do
        if [ -b "$md" ]; then
            echo "Checking $md..." >> "$report_file"
            mdadm --detail "$md" >> "$report_file"
            
            # Check for degraded arrays
            if mdadm --detail "$md" | grep -q "degraded"; then
                echo "ALERT: RAID array $md is degraded!" |
                mail -s "RAID Alert - Degraded Array" admin@example.com
            fi
        fi
    done
}
```

### Automatic RAID Monitoring
```bash
# Configure mdadm monitoring in /etc/mdadm/mdadm.conf
MAILADDR admin@example.com
MAILFROM raid-monitor@example.com
PROGRAM /usr/local/bin/raid-event-handler

# Create event handler script
cat > /usr/local/bin/raid-event-handler << 'EOF'
#!/bin/bash
event=$1
device=$2
component=$3

log_file="/var/log/raid-events.log"
echo "$(date): Event: $event Device: $device Component: $component" >> "$log_file"

case "$event" in
    "DeviceDisappeared")
        echo "RAID device $device has disappeared!" |
        mail -s "RAID Alert - Device Lost" admin@example.com
        ;;
    "Fail"|"FailSpare")
        echo "RAID component $component has failed!" |
        mail -s "RAID Alert - Component Failure" admin@example.com
        ;;
esac
EOF
chmod +x /usr/local/bin/raid-event-handler
```

## Performance Monitoring

### I/O Performance Check
```bash
#!/bin/bash
# Monitor RAID I/O performance

monitor_raid_performance() {
    local duration=$1
    local interval=5
    
    echo "Timestamp,Read_MB/s,Write_MB/s" > raid_performance.csv
    
    for ((i=0; i<duration; i+=$interval)); do
        read_speed=$(grep -A1 md0 /proc/mdstat | tail -n1 | awk '{print $4}')
        write_speed=$(grep -A1 md0 /proc/mdstat | tail -n1 | awk '{print $6}')
        echo "$(date +%s),$read_speed,$write_speed" >> raid_performance.csv
        sleep $interval
    done
}
```

### SMART Integration
```bash
#!/bin/bash
# Check SMART status of all RAID disks

check_raid_smart() {
    for md in /dev/md*; do
        if [ -b "$md" ]; then
            echo "Checking SMART status for array $md"
            mdadm --detail "$md" | grep "/dev/sd" | awk '{print $7}' | while read disk; do
                smartctl -H "$disk"
                smartctl -A "$disk" | grep -E "Reallocated_Sector|Current_Pending_Sector"
            done
        fi
    done
}
```

## Recovery Procedures

### Failed Drive Replacement
```bash
#!/bin/bash
# Script to replace failed drive

replace_failed_drive() {
    local array=$1
    local old_drive=$2
    local new_drive=$3
    
    # Mark as failed if not already
    mdadm $array --fail $old_drive
    
    # Remove the failed drive
    mdadm $array --remove $old_drive
    
    # Add the new drive
    mdadm $array --add $new_drive
    
    # Watch reconstruction
    watch -n 1 cat /proc/mdstat
}
```

## Integration with Monitoring Systems

### Prometheus Metrics
```bash
#!/bin/bash
# Generate RAID metrics for Prometheus

generate_raid_metrics() {
    echo "# HELP raid_array_state RAID array state (1 = optimal, 0 = degraded)"
    echo "# TYPE raid_array_state gauge"
    
    for md in /dev/md*; do
        if [ -b "$md" ]; then
            if mdadm --detail "$md" | grep -q "State : clean"; then
                echo "raid_array_state{device=\"$md\"} 1"
            else
                echo "raid_array_state{device=\"$md\"} 0"
            fi
        fi
    done
}
```

## Tags
#raid #storage #mdadm #system-admin