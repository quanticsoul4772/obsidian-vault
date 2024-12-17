# NVIDIA System Management Interface (nvidia-smi)

## Overview
The NVIDIA System Management Interface (nvidia-smi) is a command line utility essential for monitoring and managing NVIDIA GPU devices in high-performance computing environments. It provides comprehensive GPU monitoring, configuration, and diagnostic capabilities.

### Key Capabilities
- Real-time GPU monitoring
- Process tracking and management
- Power and performance configuration
- PCIe status monitoring
- Error reporting and diagnostics
- Thermal management

## Basic Usage

### System Overview
```bash
# Basic GPU information display
nvidia-smi

# Continuous monitoring (1-second refresh)
nvidia-smi -l 1

# CSV format for specific metrics
nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,memory.used --format=csv
```

### Process Monitoring
```bash
# View all GPU processes
nvidia-smi pmon

# Detailed process information
nvidia-smi pmon -s um -o DT
```

## Advanced Features

### PCIe Management
```bash
# Query PCIe status
nvidia-smi --query-gpu=gpu_name,pcie.link.gen.current,pcie.link.gen.max,pcie.link.width.current,pcie.link.width.max --format=csv

# Monitor PCIe bandwidth
nvidia-smi nvlink -g 0 # For GPU 0
```

### Power Management
```bash
# View power information
nvidia-smi -q -d POWER

# Set power limit (requires root)
sudo nvidia-smi -pl 200  # Sets 200W limit

# Enable persistence mode
sudo nvidia-smi -pm 1
```

### Clock Management
```bash
# Set clock speeds (requires root)
sudo nvidia-smi -ac 2505,875  # Memory clock: 2505MHz, Graphics clock: 875MHz

# Reset clocks to default
sudo nvidia-smi -rgc

# Query current clocks
nvidia-smi --query-gpu=clocks.current.graphics,clocks.current.memory --format=csv
```

## Automation Scripts

### Basic Monitoring Script
```bash
#!/bin/bash
# gpu_monitor.sh - Basic GPU monitoring script

LOG_FILE="gpu_monitoring.csv"
INTERVAL=60  # seconds

# Create header
echo "Timestamp,GPU,Temp,Utilization,Memory Used (MB),Memory Total (MB)" > $LOG_FILE

while true; do
    nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,memory.used,memory.total \
    --format=csv,nounits >> $LOG_FILE
    sleep $INTERVAL
done
```

### Performance Monitor
```bash
#!/bin/bash
# gpu_performance.sh - Detailed performance monitoring

watch -n 1 'nvidia-smi --query-gpu=timestamp,name,pstate,temperature.gpu,utilization.gpu,power.draw,fan.speed,memory.used \
--format=csv,nounits | column -t -s ","'
```

## Integration Examples

### Prometheus Integration
```bash
# Example node_exporter textfile for Prometheus
nvidia-smi --query-gpu=timestamp,name,temperature.gpu,utilization.gpu,memory.used,memory.total \
--format=csv,nounits | awk -F, '{print "nvidia_gpu_temp{gpu=\""$2"\"} "$3"\nnvidia_gpu_util{gpu=\""$2"\"} "$4}'
```

### Grafana Dashboard Queries
```sql
# GPU Temperature Query
SELECT mean("temp") FROM "nvidia_gpu" WHERE $timeFilter GROUP BY time($interval), "gpu"

# Memory Usage Query
SELECT mean("memory_used") FROM "nvidia_gpu" WHERE $timeFilter GROUP BY time($interval), "gpu"
```

## Troubleshooting Guide

### Common Issues

1. "NVIDIA-SMI has failed" error
   - Verify driver installation: `nvidia-smi -a | grep "Driver Version"`
   - Check kernel module: `lsmod | grep nvidia`
   - Review system logs: `journalctl -k | grep nvidia`

2. Performance Issues
   - Check thermal throttling: `nvidia-smi -q -d TEMPERATURE`
   - Monitor power limits: `nvidia-smi -q -d POWER`
   - Verify PCIe configuration: `nvidia-smi -q -d PCIE`

3. Memory Leaks
   - Monitor process memory: `nvidia-smi pmon -s um -o DT`
   - Track memory growth: `watch -n 1 "nvidia-smi --query-gpu=memory.used --format=csv"`

### PCIe Troubleshooting

1. Suboptimal PCIe Performance
   ```bash
   # Check current PCIe status
   nvidia-smi --query-gpu=gpu_name,pcie.link.gen.current,pcie.link.width.current --format=csv
   ```
   - PCIe 3.0 x16: ~16 GB/s bandwidth
   - PCIe 4.0 x16: ~32 GB/s bandwidth
   - If running below max capability, check:
     * BIOS PCIe settings
     * Physical slot configuration
     * Power management settings

## Best Practices

1. Monitoring
   - Implement automated monitoring with alerts
   - Log GPU metrics regularly
   - Monitor both utilization and errors
   - Track thermal trends

2. Performance Optimization
   - Use persistence mode for production servers
   - Monitor and optimize power limits
   - Track PCIe performance
   - Regular driver updates

3. Resource Management
   - Monitor process GPU memory usage
   - Track GPU utilization patterns
   - Implement proper error handling
   - Regular health checks

## Related Documentation
- [[SysAdmin - Maintaining NVIDIA Ubuntu]] - Ubuntu driver management
- [[SysAdmin - Maintaining NVIDIA Rocky]] - Rocky Linux driver management
- [[SysAdmin - NVIDIA Update Script]] - Driver update automation
- [[Doc - GPU Monitoring]] - Comprehensive monitoring guide

## Tags
#tools #nvidia #monitoring #gpu #performance #troubleshooting