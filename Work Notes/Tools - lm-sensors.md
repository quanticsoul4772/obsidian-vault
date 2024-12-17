# Hardware Monitoring with lm-sensors

## Overview
lm-sensors provides hardware health monitoring including temperature, voltage, humidity, and fan speed for Linux systems.

## Installation & Setup
```bash
# Install lm-sensors
sudo apt install lm-sensors   # Ubuntu/Debian
sudo dnf install lm-sensors   # RHEL/Rocky

# Detect sensors
sudo sensors-detect

# Apply changes
sudo systemctl restart kmod
```

## Essential Commands

### Basic Usage
```bash
# Show all sensor readings
sensors

# Show specific chip
sensors coretemp-isa-0000

# Watch readings in real-time
watch -n 1 sensors
```

### Configuration
Configuration file location: `/etc/sensors3.conf`
```bash
# Set custom thresholds
# Example configuration:
chip "coretemp-isa-0000"
    label temp1 "Core 0"
    set temp1_max 80
    set temp1_crit 90
```

## Automation Scripts

### Temperature Monitoring
```bash
#!/bin/bash
# Monitor temperatures and alert if threshold exceeded
MAX_TEMP=80
ALERT_EMAIL="admin@example.com"

check_temps() {
    sensors -u | grep -A2 temp1_input | while read line; do
        if [[ $line =~ temp1_input:\ +([0-9]+\.[0-9]+) ]]; then
            temp=${BASH_REMATCH[1]}
            if (( $(echo "$temp > $MAX_TEMP" | bc -l) )); then
                echo "Temperature Alert: ${temp}°C" |
                mail -s "High Temperature Alert" "$ALERT_EMAIL"
            fi
        fi
    done
}
```

### Data Collection Script
```bash
#!/bin/bash
# Collect and store sensor data in JSON format
OUTPUT_DIR="/var/log/sensors"
mkdir -p "$OUTPUT_DIR"

collect_sensor_data() {
    timestamp=$(date +"%Y%m%d_%H%M%S")
    sensors -j > "$OUTPUT_DIR/sensors_${timestamp}.json"
}

# Run every 5 minutes via crontab:
# */5 * * * * /path/to/collect_sensor_data.sh
```

## Integration with Monitoring Systems

### Prometheus Integration
```bash
# Install node_exporter with sensors collection
apt install prometheus-node-exporter

# Enable sensors collector
systemctl edit prometheus-node-exporter
# Add:
[Service]
Environment="EXTRA_ARGS=--collector.textfile.directory=/var/lib/node_exporter/textfile --collector.sensors"
```

### Grafana Dashboard
```json
{
  "panels": [
    {
      "title": "CPU Temperature",
      "type": "graph",
      "query": "node_thermal_zone_temp"
    },
    {
      "title": "Fan Speeds",
      "type": "graph",
      "query": "node_fan_speed_rpm"
    }
  ]
}
```

## Tags
#monitoring #hardware #sensors #temperature