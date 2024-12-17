# NVIDIA Data Center GPU Manager (DCGM)

## Overview
NVIDIA DCGM is a suite of tools providing system administrators with comprehensive monitoring, management, and diagnostics capabilities for NVIDIA data center GPUs. It's particularly valuable in high-performance computing environments for health monitoring, diagnostics, and policy-based governance.

### Key Features
- Health and diagnostics monitoring
- Process statistics tracking
- Policy management
- Configuration management
- Active health diagnostics
- Job statistics collection
- Remote management capabilities

## Basic Usage

### Installation
```bash
# For Ubuntu/Debian
sudo apt-get update
sudo apt-get install datacenter-gpu-manager

# For RHEL/Rocky Linux
sudo dnf install epel-release
sudo dnf install datacenter-gpu-manager
```

### Service Management
```bash
# Start DCGM service
sudo systemctl start nvidia-dcgm

# Enable DCGM at boot
sudo systemctl enable nvidia-dcgm

# Check service status
sudo systemctl status nvidia-dcgm
```

### Basic Commands
```bash
# Check DCGM status
dcgmi status

# Get GPU information
dcgmi discovery -l

# Run health checks
dcgmi diag -r 1

# Show GPU statistics
dcgmi dmon
```

## Advanced Features

### Health Monitoring
```bash
# Run comprehensive health check
dcgmi diag -r 3

# Monitor specific GPU
dcgmi health -g 0

# Set up watch policies
dcgmi policy -g 0 --set 0,0,0,80 # Sets temperature threshold to 80°C
```

### Profiling and Statistics
```bash
# Start job stats collection
dcgmi stats -g 0 -e

# View job statistics
dcgmi stats -g 0 -v

# Export statistics to file
dcgmi stats -x stats.json
```

### Configuration Management
```bash
# Set compute mode
dcgmi config -g 0 --set -c 0

# Set power limit
dcgmi config -g 0 --set -p 250

# Enable persistence mode
dcgmi config -g 0 --set -m 1
```

## Automation Scripts

### Health Check Script
```bash
#!/bin/bash
# dcgm_health_monitor.sh

LOG_DIR="/var/log/dcgm"
ALERT_EMAIL="admin@example.com"

mkdir -p $LOG_DIR

check_gpu_health() {
    local TIMESTAMP=$(date +%Y%m%d-%H%M%S)
    local LOG_FILE="$LOG_DIR/health_check_$TIMESTAMP.log"
    
    # Run health check
    dcgmi diag -r 3 > "$LOG_FILE"
    
    # Check for failures
    if grep -i "fail" "$LOG_FILE" > /dev/null; then
        # Send alert
        mail -s "DCGM Health Check Alert" "$ALERT_EMAIL" < "$LOG_FILE"
    fi
}

# Run health check every 6 hours
while true; do
    check_gpu_health
    sleep 21600
done
```

### Performance Monitoring Script
```bash
#!/bin/bash
# dcgm_performance_monitor.sh

LOG_DIR="/var/log/dcgm/performance"
INTERVAL=300  # 5 minutes

mkdir -p $LOG_DIR

monitor_performance() {
    local TIMESTAMP=$(date +%Y%m%d-%H%M%S)
    
    # Collect GPU metrics
    dcgmi dmon -e 155,150,203,204,206,207 -c 1 > \
        "$LOG_DIR/perf_metrics_$TIMESTAMP.log"
    
    # Process statistics collection
    dcgmi stats -v >> "$LOG_DIR/stats_$TIMESTAMP.log"
}

while true; do
    monitor_performance
    sleep $INTERVAL
done
```

## Integration Examples

### Prometheus Integration
```yaml
# /etc/prometheus/prometheus.yml
scrape_configs:
  - job_name: 'dcgm_exporter'
    static_configs:
      - targets: ['localhost:9400']
```

### Docker Integration
```dockerfile
# Dockerfile for DCGM-enabled container
FROM nvidia/cuda:11.8.0-base-ubuntu22.04

# Install DCGM
RUN apt-get update && apt-get install -y --no-install-recommends \
    datacenter-gpu-manager \
    && rm -rf /var/lib/apt/lists/*

# Start DCGM service
CMD ["dcgmi", "agent", "-d"]
```

## Troubleshooting Guide

### Common Issues

1. Service Start Failure
   - Check system requirements
   ```bash
   dcgmi discovery -l
   nvidia-smi
   ```
   - Verify driver compatibility
   ```bash
   dcgmi discovery --drivers
   ```

2. Health Check Failures
   - Review specific test results
   ```bash
   dcgmi diag -r 3 -v
   ```
   - Check GPU temperature and power
   ```bash
   dcgmi dmon -e 155,150
   ```

3. Monitoring Issues
   - Verify DCGM agent status
   ```bash
   dcgmi agent -s
   ```
   - Check connection to GPUs
   ```bash
   dcgmi discovery -l
   ```

## Best Practices

1. Monitoring Setup
   - Configure regular health checks
   - Set up automated alerts
   - Maintain historical data
   - Monitor all critical metrics

2. Performance Optimization
   - Regular profiling
   - Policy-based management
   - Resource utilization tracking
   - Proactive issue detection

3. Security Considerations
   - Role-based access control
   - Secure communication
   - Regular updates
   - Audit logging

4. Integration Guidelines
   - Containerization support
   - Monitoring stack integration
   - Job scheduler integration
   - Custom tool development

## Related Documentation
- [[Tools - GPU Monitoring and Management Guide]] - Comprehensive monitoring guide
- [[Tools - nvidia-smi Consolidated]] - NVIDIA SMI reference
- [[SysAdmin - NVIDIA Update Script]] - Driver management
- [[Troubleshooting Decision Tree]] - Systematic problem resolution

## Tags
#dcgm #nvidia #monitoring #gpu #management #hpc