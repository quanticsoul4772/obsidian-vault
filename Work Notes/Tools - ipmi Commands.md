# IPMI Commands Reference

## Overview
IPMI (Intelligent Platform Management Interface) provides hardware-level access to server management features.

## Essential Commands

### System Information
```bash
# Get system information
ipmitool fru print

# Get sensor readings
ipmitool sensor list

# Get SEL (System Event Log)
ipmitool sel list
```

### Power Management
```bash
# Get power status
ipmitool power status

# Power operations
ipmitool power on
ipmitool power off
ipmitool power cycle
ipmitool power reset
```

### Temperature and Fan Control
```bash
# Get temperature readings
ipmitool sdr type temperature

# Get fan speeds
ipmitool sdr type fan

# Set fan speed (if supported)
ipmitool raw 0x30 0x70 0x66 0x01 0x00 SPEED
```

### Chassis Control
```bash
# Get chassis status
ipmitool chassis status

# Identify LED control
ipmitool chassis identify 15  # Turn on LED for 15 seconds
```

### Network Configuration
```bash
# Get LAN configuration
ipmitool lan print 1

# Set IP address
ipmitool lan set 1 ipaddr x.x.x.x
```

## Automation Scripts

### System Health Check
```bash
#!/bin/bash
# Collect system health information
timestamp=$(date +"%Y%m%d_%H%M%S")
output_dir="ipmi_health_${timestamp}"
mkdir -p "$output_dir"

# Collect various IPMI data
ipmitool sel list > "$output_dir/sel.log"
ipmitool sensor list > "$output_dir/sensors.log"
ipmitool sdr type temperature > "$output_dir/temperature.log"
ipmitool sdr type fan > "$output_dir/fans.log"
ipmitool fru print > "$output_dir/fru.log"
ipmitool chassis status > "$output_dir/chassis.log"

# Parse and check for critical conditions
grep -i "critical" "$output_dir/sensors.log" > "$output_dir/critical_sensors.log"
grep -i "warning" "$output_dir/sel.log" > "$output_dir/warnings.log"
```

### Temperature Monitoring
```bash
#!/bin/bash
# Monitor temperatures and alert if threshold exceeded
MAX_TEMP=80
ALERT_EMAIL="admin@example.com"

temp_check() {
    ipmitool sdr type temperature | while read line; do
        sensor=$(echo $line | cut -d'|' -f1)
        temp=$(echo $line | cut -d'|' -f2 | grep -oE '[0-9]+')
        if [ "$temp" -gt "$MAX_TEMP" ]; then
            echo "High temperature alert: $sensor at ${temp}C" | 
            mail -s "Temperature Alert" "$ALERT_EMAIL"
        fi
    done
}

temp_check
```

## Integration with Other Tools

### Monitoring Integration
- Works with Nagios/Icinga for alerts
- Can be integrated with Prometheus using node_exporter
- Supports SNMP for broader monitoring systems

### Automation Integration
```bash
#!/bin/bash
# Example of integrating with other monitoring tools
# Collect IPMI data in JSON format

collect_ipmi_data() {
    echo "{"
    echo "  \"temperature\": ["
    ipmitool sdr type temperature | \
    awk -F'|' '{printf "    {\"sensor\": \"%s\", \"reading\": \"%s\", \"status\": \"%s\"},\n", $1, $2, $3}'
    echo "  ],"
    echo "  \"fans\": ["
    ipmitool sdr type fan | \
    awk -F'|' '{printf "    {\"sensor\": \"%s\", \"reading\": \"%s\", \"status\": \"%s\"},\n", $1, $2, $3}'
    echo "  ]"
    echo "}"
} > ipmi_data.json
```

## Troubleshooting

### Common Issues
1. Connection Issues
```bash
# Test IPMI connectivity
ipmitool -I lanplus -H [BMC IP] -U [username] -P [password] chassis status
```

2. Sensor Reading Errors
```bash
# Reset BMC
ipmitool mc reset cold
```

3. Authentication Failures
```bash
# Check user list
ipmitool user list 1
```

## Tags
#ipmi #server-management #monitoring #automation