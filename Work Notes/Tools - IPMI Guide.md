# IPMI Command Reference Guide

## Overview
IPMI (Intelligent Platform Management Interface) provides hardware-level access to server management features. This guide covers the utility `ipmitool`, which is used for managing and configuring devices that support the IPMI protocol.

## Basic Syntax
```bash
ipmitool [options] <command>

# Common Options
-I <interface>    # Specify interface (lan, lanplus, open)
-H <hostname>     # Remote host name or IP address
-U <username>     # Remote username
-P <password>     # Remote password
-L <priv_level>   # Remote session privilege level
-p <port>         # Remote RMCP port (default 623)

# Remote command example
ipmitool -I lanplus -H 192.168.1.100 -U admin -P password power status
```

## Essential Commands

### Power Management
```bash
# Get power status
ipmitool power status

# Power operations
ipmitool power on
ipmitool power off
ipmitool power cycle
ipmitool power soft      # Soft shutdown
ipmitool power reset
```

### System Information
```bash
# Basic system info
ipmitool fru print

# BMC info
ipmitool mc info

# Chassis status
ipmitool chassis status
```

### Sensor Management
```bash
# All sensor readings
ipmitool sensor list

# Specific sensor reading
ipmitool sensor get "CPU Temp"

# Temperature sensors
ipmitool sdr type temperature

# Fan sensors
ipmitool sdr type fan

# Set fan speed (if supported)
ipmitool raw 0x30 0x70 0x66 0x01 0x00 SPEED
```

### System Event Log (SEL)
```bash
# View SEL
ipmitool sel list

# Clear SEL
ipmitool sel clear

# Save SEL to file
ipmitool sel save sel.log
```

### Boot Control
```bash
# Set next boot device
ipmitool chassis bootdev pxe     # Boot to PXE
ipmitool chassis bootdev disk    # Boot to disk
ipmitool chassis bootdev cdrom   # Boot to CD/DVD
ipmitool chassis bootdev bios    # Boot to BIOS
```

### User Management
```bash
# List users
ipmitool user list 1

# Set user password
ipmitool user set password 2 mypassword

# Add new user
ipmitool user set name 3 newuser
ipmitool user set password 3 password
ipmitool user priv 3 4           # Set privilege level
ipmitool user enable 3           # Enable user
```

### Network Configuration
```bash
# Show network settings
ipmitool lan print 1

# Set network parameters
ipmitool lan set 1 ipaddr 192.168.1.100
ipmitool lan set 1 netmask 255.255.255.0
ipmitool lan set 1 gateway 192.168.1.1
```

### Serial Over LAN (SOL)
```bash
# Activate SOL
ipmitool sol activate

# Deactivate SOL
ipmitool sol deactivate

# Configure SOL
ipmitool sol set volatile-bit-rate 115.2     # Set baud rate
ipmitool sol set privilege-level admin       # Set privilege level
```

## Automation Scripts

### System Health Monitoring
```bash
#!/bin/bash
# Comprehensive health check script

# Configuration
LOG_DIR="/var/log/ipmi"
ALERT_EMAIL="admin@example.com"
TEMP_THRESHOLD=80
HOSTS_FILE="/etc/ipmi/hosts.txt"

mkdir -p "$LOG_DIR"

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_DIR/health_check.log"
}

check_host() {
    local host="$1"
    local user="$2"
    local pass="$3"
    local timestamp=$(date +"%Y%m%d_%H%M%S")
    local host_dir="$LOG_DIR/$host/$timestamp"
    
    mkdir -p "$host_dir"
    
    # Collect system data
    ipmitool -I lanplus -H "$host" -U "$user" -P "$pass" sel list > "$host_dir/sel.log"
    ipmitool -I lanplus -H "$host" -U "$user" -P "$pass" sensor list > "$host_dir/sensors.log"
    ipmitool -I lanplus -H "$host" -U "$user" -P "$pass" chassis status > "$host_dir/chassis.log"
    
    # Check for critical conditions
    if grep -i "critical" "$host_dir/sensors.log"; then
        log_message "CRITICAL: Found critical sensor readings on $host"
        mail -s "Critical Sensor Alert - $host" "$ALERT_EMAIL" < "$host_dir/sensors.log"
    fi
    
    # Check temperatures
    while read -r line; do
        if [[ $line =~ "Temperature" ]]; then
            temp=$(echo $line | awk '{print $4}' | cut -d. -f1)
            if [ "$temp" -gt "$TEMP_THRESHOLD" ]; then
                log_message "WARNING: High temperature detected on $host: $temp°C"
                echo "High temperature alert: $line" | mail -s "Temperature Alert - $host" "$ALERT_EMAIL"
            fi
        fi
    done < "$host_dir/sensors.log"
}

# Main execution
while IFS=, read -r host user pass; do
    [ -z "$host" ] && continue
    log_message "Checking host: $host"
    check_host "$host" "$user" "$pass"
done < "$HOSTS_FILE"
```

### Power Management Automation
```bash
#!/bin/bash
# Automated power management script

# Configuration
CONFIG_FILE="/etc/ipmi/power_schedule.conf"
LOG_FILE="/var/log/ipmi/power_management.log"

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOG_FILE"
}

manage_power() {
    local host="$1"
    local action="$2"
    local result
    
    case "$action" in
        "on"|"off"|"cycle"|"reset")
            result=$(ipmitool -I lanplus -H "$host" power "$action" 2>&1)
            log_message "Power $action on $host: $result"
            ;;
        *)
            log_message "Invalid power action: $action"
            return 1
            ;;
    esac
}

# Read schedule and execute
while IFS=, read -r host action time; do
    [ -z "$host" ] && continue
    current_time=$(date +%H:%M)
    if [ "$current_time" = "$time" ]; then
        manage_power "$host" "$action"
    fi
done < "$CONFIG_FILE"
```

## Integration Examples

### Prometheus Integration
```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'ipmi_exporter'
    static_configs:
      - targets: ['localhost:9290']
    metrics_path: '/metrics'
    params:
      module: ['default']

# Alert rules
groups:
  - name: ipmi_alerts
    rules:
      - alert: IPMIHighTemperature
        expr: ipmi_temperature_celsius > 80
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High temperature detected"
```

## Troubleshooting Guide

### Common Issues

1. Connection Problems
```bash
# Test connectivity
ping <BMC_IP>
nc -zv <BMC_IP> 623

# Check BMC status
ipmitool mc info

# Reset BMC
ipmitool mc reset cold
```

2. Authentication Issues
```bash
# Verify user credentials
ipmitool user list 1

# Check privilege level
ipmitool channel getciphers ipmi

# Test with different authentication types
ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> -L ADMINISTRATOR chassis status
```

3. Sensor Reading Errors
```bash
# Reinitialize sensors
ipmitool sensor list

# Check SDR cache
ipmitool sdr dump sdr.cache
ipmitool -S sdr.cache sensor list
```

## Best Practices

1. Security
   - Use lanplus interface for encryption
   - Implement strong passwords
   - Regularly audit user access
   - Monitor failed login attempts

2. Monitoring
   - Regular health checks
   - Temperature monitoring
   - Event log review
   - Performance tracking

3. Maintenance
   - Regular BMC firmware updates
   - Configuration backups
   - Log rotation
   - Documentation updates

4. Automation
   - Scheduled health checks
   - Automated reporting
   - Alert configuration
   - Power management schedules

## Tags
#ipmi #server-management #monitoring #automation #troubleshooting