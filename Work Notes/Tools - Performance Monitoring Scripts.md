# System Performance Monitoring Scripts

## Overview
This guide provides a collection of performance monitoring scripts for system administrators managing HPC and data center environments. These scripts focus on comprehensive system monitoring, including CPU, memory, disk, network, and GPU resources.

### Key Features
- Real-time performance monitoring
- Resource utilization tracking
- Health checks and alerting
- Data collection for trend analysis
- Integration with monitoring systems

## Basic Monitoring Scripts

### System Resource Monitor
```bash
#!/bin/bash
# system_monitor.sh
# Monitors CPU, memory, disk, and system load

LOG_DIR="/var/log/monitoring"
INTERVAL=300  # 5 minutes

mkdir -p "$LOG_DIR"

monitor_system() {
    local TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
    local LOG_FILE="$LOG_DIR/system_metrics.log"
    
    # CPU Usage
    local CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
    
    # Memory Usage
    local MEM_INFO=$(free -m | grep Mem)
    local MEM_TOTAL=$(echo "$MEM_INFO" | awk '{print $2}')
    local MEM_USED=$(echo "$MEM_INFO" | awk '{print $3}')
    local MEM_PERCENTAGE=$(awk "BEGIN {print ($MEM_USED/$MEM_TOTAL)*100}")
    
    # Disk Usage
    local DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}' | sed 's/%//')
    
    # System Load
    local LOAD_AVG=$(uptime | awk -F'load average:' '{print $2}' | tr -d ' ')
    
    echo "$TIMESTAMP,CPU=$CPU_USAGE%,MEM=$MEM_PERCENTAGE%,DISK=$DISK_USAGE%,LOAD=$LOAD_AVG" >> "$LOG_FILE"
}

# Main loop
while true; do
    monitor_system
    sleep "$INTERVAL"
done
```

### Process Monitor
```bash
#!/bin/bash
# process_monitor.sh
# Tracks resource usage of specific processes

LOG_DIR="/var/log/monitoring/processes"
INTERVAL=60  # 1 minute

# List of processes to monitor
PROCESSES=(
    "nginx"
    "postgresql"
    "mongodb"
)

mkdir -p "$LOG_DIR"

monitor_process() {
    local PROCESS=$1
    local TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
    local LOG_FILE="$LOG_DIR/${PROCESS}_metrics.log"
    
    # Get process statistics
    if pgrep "$PROCESS" > /dev/null; then
        local CPU=$(ps aux | grep "$PROCESS" | grep -v grep | awk '{sum+=$3} END {print sum}')
        local MEM=$(ps aux | grep "$PROCESS" | grep -v grep | awk '{sum+=$4} END {print sum}')
        local THREADS=$(ps -eLf | grep "$PROCESS" | grep -v grep | wc -l)
        
        echo "$TIMESTAMP,CPU=$CPU%,MEM=$MEM%,THREADS=$THREADS" >> "$LOG_FILE"
    fi
}

# Main loop
while true; do
    for PROCESS in "${PROCESSES[@]}"; do
        monitor_process "$PROCESS"
    done
    sleep "$INTERVAL"
done
```

### Network Performance Monitor
```bash
#!/bin/bash
# network_monitor.sh
# Monitors network interface performance

LOG_DIR="/var/log/monitoring/network"
INTERVAL=300  # 5 minutes
INTERFACE="eth0"  # Change to match your primary interface

mkdir -p "$LOG_DIR"

monitor_network() {
    local TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
    local LOG_FILE="$LOG_DIR/network_metrics.log"
    
    # Get network statistics
    local STATS=$(ip -s link show "$INTERFACE")
    local RX_BYTES=$(echo "$STATS" | grep "RX:" -A 1 | tail -1 | awk '{print $1}')
    local TX_BYTES=$(echo "$STATS" | grep "TX:" -A 1 | tail -1 | awk '{print $1}')
    
    # Calculate bandwidth (bytes per second)
    if [ -f "$LOG_FILE.tmp" ]; then
        local PREV_STATS=$(tail -1 "$LOG_FILE.tmp")
        local PREV_RX=$(echo "$PREV_STATS" | cut -d',' -f4)
        local PREV_TX=$(echo "$PREV_STATS" | cut -d',' -f5)
        
        local RX_BPS=$(( (RX_BYTES - PREV_RX) / INTERVAL ))
        local TX_BPS=$(( (TX_BYTES - PREV_TX) / INTERVAL ))
        
        echo "$TIMESTAMP,RX_BPS=$RX_BPS,TX_BPS=$TX_BPS,RX_TOTAL=$RX_BYTES,TX_TOTAL=$TX_BYTES" >> "$LOG_FILE"
    fi
    
    echo "$TIMESTAMP,0,0,$RX_BYTES,$TX_BYTES" > "$LOG_FILE.tmp"
}

# Main loop
while true; do
    monitor_network
    sleep "$INTERVAL"
done
```

## Advanced Monitoring Scripts

### Comprehensive System Monitor
```bash
#!/bin/bash
# comprehensive_monitor.sh
# Advanced system monitoring with alerting and data export

# Configuration
LOG_DIR="/var/log/monitoring"
ALERT_EMAIL="admin@example.com"
INTERVAL=300
ALERT_THRESHOLDS=(
    "CPU:90"      # CPU usage threshold (%)
    "MEM:85"      # Memory usage threshold (%)
    "DISK:90"     # Disk usage threshold (%)
    "LOAD:10"     # Load average threshold
)

# Initialize
mkdir -p "$LOG_DIR"
declare -A ALERTS_SENT

# Monitoring functions
check_thresholds() {
    local METRIC=$1
    local VALUE=$2
    local THRESHOLD=$3
    local TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
    
    if (( $(echo "$VALUE > $THRESHOLD" | bc -l) )); then
        # Avoid alert spam - only alert once per hour
        local LAST_ALERT=${ALERTS_SENT["$METRIC"]}
        local CURRENT_TIME=$(date +%s)
        
        if [ -z "$LAST_ALERT" ] || [ $((CURRENT_TIME - LAST_ALERT)) -gt 3600 ]; then
            echo "Alert: $METRIC usage at $VALUE% (threshold: $THRESHOLD%)" | \
                mail -s "System Alert: High $METRIC Usage" "$ALERT_EMAIL"
            ALERTS_SENT["$METRIC"]=$CURRENT_TIME
        fi
    fi
}

monitor_all() {
    local TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
    local LOG_FILE="$LOG_DIR/comprehensive_metrics.csv"
    
    # System metrics
    local CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}')
    local MEM_INFO=$(free -m | grep Mem)
    local MEM_PERCENTAGE=$(echo "$MEM_INFO" | awk '{print ($3/$2)*100}')
    local DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}' | sed 's/%//')
    local LOAD_AVG=$(uptime | awk -F'load average:' '{print $2}' | cut -d, -f1 | tr -d ' ')
    
    # Network metrics
    local NETSTAT=$(netstat -i | grep "$INTERFACE")
    local RX_PACKETS=$(echo "$NETSTAT" | awk '{print $3}')
    local TX_PACKETS=$(echo "$NETSTAT" | awk '{print $7}')
    
    # Process count
    local PROCESS_COUNT=$(ps aux | wc -l)
    
    # IO stats
    local IO_STATS=$(iostat -x 1 1 | grep sda)
    local IO_UTIL=$(echo "$IO_STATS" | awk '{print $14}')
    
    # Log data
    echo "$TIMESTAMP,$CPU_USAGE,$MEM_PERCENTAGE,$DISK_USAGE,$LOAD_AVG,$RX_PACKETS,$TX_PACKETS,$PROCESS_COUNT,$IO_UTIL" >> "$LOG_FILE"
    
    # Check thresholds
    for threshold in "${ALERT_THRESHOLDS[@]}"; do
        local METRIC=$(echo "$threshold" | cut -d: -f1)
        local VALUE=$(echo "$threshold" | cut -d: -f2)
        
        case "$METRIC" in
            "CPU")  check_thresholds "CPU" "$CPU_USAGE" "$VALUE" ;;
            "MEM")  check_thresholds "Memory" "$MEM_PERCENTAGE" "$VALUE" ;;
            "DISK") check_thresholds "Disk" "$DISK_USAGE" "$VALUE" ;;
            "LOAD") check_thresholds "Load" "$LOAD_AVG" "$VALUE" ;;
        esac
    done
}

# Data export function
export_to_prometheus() {
    # Create Prometheus-format metrics
    local METRIC_FILE="/var/lib/node_exporter/custom_metrics.prom"
    
    {
        echo "# HELP system_cpu_usage Current CPU usage"
        echo "# TYPE system_cpu_usage gauge"
        echo "system_cpu_usage $CPU_USAGE"
        
        echo "# HELP system_memory_usage Current memory usage"
        echo "# TYPE system_memory_usage gauge"
        echo "system_memory_usage $MEM_PERCENTAGE"
    } > "$METRIC_FILE"
}

# Main loop
while true; do
    monitor_all
    export_to_prometheus
    sleep "$INTERVAL"
done
```

## Integration Examples

### Prometheus Integration
```yaml
# prometheus.yml configuration for custom metrics
scrape_configs:
  - job_name: 'custom_monitoring'
    static_configs:
      - targets: ['localhost:9100']
    file_sd_configs:
      - files:
        - '/var/lib/node_exporter/custom_metrics.prom'
```

### Grafana Dashboard
```json
{
  "dashboard": {
    "panels": [
      {
        "title": "System Resource Usage",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "system_cpu_usage",
            "legendFormat": "CPU Usage"
          },
          {
            "expr": "system_memory_usage",
            "legendFormat": "Memory Usage"
          }
        ]
      }
    ]
  }
}
```

## Best Practices

### 1. Resource Efficiency
- Use appropriate monitoring intervals
- Implement log rotation
- Clean up temporary files
- Optimize data collection methods

### 2. Data Management
- Regular log archival
- Data compression
- Retention policies
- Backup monitoring data

### 3. Alert Management
- Define clear thresholds
- Avoid alert fatigue
- Implement escalation procedures
- Document alert responses

### 4. Script Maintenance
- Version control
- Regular testing
- Performance optimization
- Documentation updates

## Troubleshooting Guide

### Common Issues

1. High Resource Usage
   - Check monitoring interval
   - Verify log rotation
   - Optimize data collection

2. Missing Data
   - Check file permissions
   - Verify disk space
   - Monitor process status

3. Alert Issues
   - Verify email configuration
   - Check threshold values
   - Test alert mechanisms

## Related Documentation
- [[Tools - GPU Monitoring and Management Guide]]
- [[Tools - DCGM]]
- [[Tools - nvidia-smi Consolidated]]
- [[Troubleshooting Decision Tree]]

## Tags
#monitoring #performance #scripts #automation #system-admin