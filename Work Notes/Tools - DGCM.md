# Data Center GPU Manager (DGCM)

## Overview
DGCM is a suite of tools for managing and monitoring NVIDIA GPUs in data center environments. It provides automation capabilities, health monitoring, and cluster-wide GPU management.

## Installation
```bash
# Install DGCM
sudo apt install datacenter-gpu-manager    # Ubuntu
sudo yum install datacenter-gpu-manager    # RHEL/Rocky

# Enable and start service
sudo systemctl enable nvidia-dcgm
sudo systemctl start nvidia-dcgm
```

## Basic Commands

### System Health Checks
```bash
# Check GPU health
dcgmi health -g 1

# Run diagnostic test
dcgmi diag -r 1

# Check all GPUs status
dcgmi discovery -l
```

### Monitoring Commands
```bash
# Get GPU stats
dcgmi stats -e

# Watch GPU metrics
dcgmi dmon

# Get power usage
dcgmi profile -g 1
```

## Automation Scripts

### GPU Health Monitor
```bash
#!/bin/bash
# Monitor GPU health and report issues

check_gpu_health() {
    local log_file="/var/log/gpu_health.log"
    local alert_email="admin@example.com"
    
    echo "GPU Health Check - $(date)" >> "$log_file"
    
    # Run health check
    dcgmi health -g 1 > temp_health.txt
    
    # Check for issues
    if grep -q "Error" temp_health.txt; then
        cat temp_health.txt | mail -s "GPU Health Alert" "$alert_email"
    fi
    
    # Log results
    cat temp_health.txt >> "$log_file"
    rm temp_health.txt
}
```

### Performance Monitoring
```bash
#!/bin/bash
# Collect GPU performance metrics

monitor_gpu_performance() {
    local output_dir="/var/log/gpu_metrics"
    mkdir -p "$output_dir"
    
    # Collection interval (seconds)
    interval=60
    
    while true; do
        timestamp=$(date +"%Y%m%d_%H%M%S")
        dcgmi stats -e > "$output_dir/stats_${timestamp}.json"
        sleep $interval
    done
}
```

## Configuration Management

### Group Management
```bash
# Create GPU group
dcgmi group -c gpugroup1 -g 1

# Add GPUs to group
dcgmi group -a $gpu_id -g 1

# Set policies for group
dcgmi policy -g 1 -s p
```

### Profile Management
```bash
# Create compute profile
dcgmi profile -g 1 --enforce compute

# Create custom profile
cat << EOF > custom_profile.json
{
    "version": "1.0",
    "settings": {
        "power_limit": 250,
        "max_memory_clock": 5001,
        "max_gpu_clock": 1380
    }
}
EOF
dcgmi profile -g 1 --enforce custom_profile.json
```

## Integration Examples

### Prometheus Integration
```bash
#!/bin/bash
# Generate metrics for Prometheus

generate_gpu_metrics() {
    echo "# HELP dcgm_gpu_temp GPU temperature in celsius"
    echo "# TYPE dcgm_gpu_temp gauge"
    
    dcgmi stats -e | jq -r '.["GPU Stats"][] | 
        "dcgm_gpu_temp{gpu=\"\(.gpu_id)\"} \(.temperature)"'
}

# Save to node_exporter directory
generate_gpu_metrics > /var/lib/node_exporter/gpu_metrics.prom
```

### Grafana Dashboard
```json
{
  "dashboard": {
    "panels": [
      {
        "title": "GPU Temperature",
        "type": "graph",
        "targets": [
          {
            "expr": "dcgm_gpu_temp",
            "legendFormat": "GPU {{gpu}}"
          }
        ]
      },
      {
        "title": "GPU Utilization",
        "type": "graph",
        "targets": [
          {
            "expr": "dcgm_gpu_util",
            "legendFormat": "GPU {{gpu}}"
          }
        ]
      }
    ]
  }
}
```

## Diagnostic Procedures

### Common Issues Resolution
```bash
#!/bin/bash
# GPU diagnostic and resolution script

diagnose_gpu() {
    local gpu_id=$1
    local report_file="gpu_diagnostic_$(date +%Y%m%d_%H%M%S).log"
    
    echo "GPU Diagnostic Report - GPU $gpu_id" > "$report_file"
    echo "=================================" >> "$report_file"
    
    # Check GPU health
    dcgmi health -g 1 >> "$report_file"
    
    # Run diagnostics
    dcgmi diag -r 1 >> "$report_file"
    
    # Check memory errors
    dcgmi stats -e | grep -A 5 "Memory Errors" >> "$report_file"
    
    # Temperature history
    dcgmi stats -j >> "$report_file"
}
```

### Performance Validation
```bash
#!/bin/bash
# Validate GPU performance

validate_performance() {
    local gpu_id=$1
    local threshold=0.9  # 90% of expected performance
    
    # Run performance test
    dcgmi diag -r 3 > perf_test.txt
    
    # Check results
    if grep -q "Failed" perf_test.txt; then
        echo "Performance validation failed for GPU $gpu_id"
        return 1
    fi
    
    return 0
}
```

## Best Practices

### Regular Maintenance
1. Schedule health checks:
```bash
# Add to crontab
0 */4 * * * /usr/local/bin/check_gpu_health.sh
```

2. Monitor power efficiency:
```bash
#!/bin/bash
# Monitor power efficiency
dcgmi profile -g 1 --get
```

3. Temperature management:
```bash
#!/bin/bash
# Adjust GPU settings based on temperature
monitor_temp() {
    local max_temp=80
    
    while true; do
        current_temp=$(dcgmi stats -e | jq '.temperature')
        if [ "$current_temp" -gt "$max_temp" ]; then
            dcgmi profile -g 1 --enforce power_saving
        fi
        sleep 300
    done
}
```

## Tags
#gpu #nvidia #monitoring #datacenter #cluster-management