# GPU Monitoring and Management Tools Guide

## Overview
This guide provides a comprehensive overview of tools and strategies for GPU monitoring and management in high-performance computing environments. It covers vendor-specific tools, open-source solutions, and integration strategies for effective GPU resource management.

## Tool Categories

### NVIDIA Tools
1. **NVIDIA Data Center GPU Manager (DCGM)**
   - Advanced monitoring and management
   - Health monitoring and diagnostics
   - Telemetry and policy-based governance
   - See: [[Tools - DGCM]]

2. **NVIDIA System Management Interface**
   - Command-line monitoring and management
   - Process and resource tracking
   - See: [[Tools - nvidia-smi Consolidated]]

3. **Development and Profiling**
   - Nsight Systems: System-wide performance analysis
   - Nsight Compute: CUDA kernel profiling
   - NVIDIA Visual Profiler (Legacy)

### Open-Source Solutions

1. **System Monitoring**
   ```bash
   # Prometheus NVIDIA GPU Exporter
   docker run -d --gpus all nvidia/dcgm-exporter

   # Collectd NVIDIA Plugin Configuration
   LoadPlugin nvidia
   <Plugin nvidia>
       Report "GPU_TEMP"
       Report "MEMORY_USED"
       Report "GPU_UTIL"
   </Plugin>
   ```

2. **Visualization Tools**
   - Grafana: GPU dashboard creation
   - Netdata: Real-time monitoring
   - Custom solutions using pynvml

### Cluster Management Integration

1. **Slurm Configuration**
   ```bash
   # GPU Resource Definition
   GresTypes=gpu
   NodeName=node[1-4] Gres=gpu:nvidia:4
   
   # Job Submission with GPU
   sbatch --gres=gpu:2 myjob.sh
   ```

2. **Other Job Schedulers**
   - IBM Spectrum LSF
   - Altair PBS Professional
   - Platform-specific configurations

## Automation Scripts

### 1. Basic GPU Monitoring
```bash
#!/bin/bash
# comprehensive_gpu_monitor.sh

# Required tools: nvidia-smi, dcgm-exporter (optional)
LOG_DIR="/var/log/gpu-monitoring"
INTERVAL=300  # 5 minutes

mkdir -p $LOG_DIR

while true; do
    TIMESTAMP=$(date +%Y%m%d-%H%M%S)
    
    # Basic GPU info
    nvidia-smi --query-gpu=timestamp,name,pci.bus_id,driver_version,pstate,temperature.gpu,utilization.gpu,utilization.memory,memory.used,memory.total --format=csv > "$LOG_DIR/gpu_status_$TIMESTAMP.csv"
    
    # Process information
    nvidia-smi pmon -s um -o DT > "$LOG_DIR/gpu_processes_$TIMESTAMP.log"
    
    # Error checking
    nvidia-smi -q -d ERROR >> "$LOG_DIR/gpu_errors_$TIMESTAMP.log"
    
    sleep $INTERVAL
done
```

### 2. Health Check Script
```bash
#!/bin/bash
# gpu_health_check.sh

check_gpu_health() {
    # Temperature check
    TEMP=$(nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader)
    if [ $TEMP -gt 80 ]; then
        echo "WARNING: GPU temperature is $TEMP°C"
    fi
    
    # Memory check
    nvidia-smi --query-gpu=memory.used,memory.total --format=csv,noheader | awk -F, '{
        used=$1;
        total=$2;
        percentage=(used/total)*100;
        if (percentage > 90) {
            print "WARNING: Memory usage is " percentage "%"
        }
    }'
    
    # Error check
    if nvidia-smi -q -d ERROR | grep -i "error" > /dev/null; then
        echo "ERROR: GPU errors detected"
    fi
}

check_gpu_health
```

## Integration Examples

### 1. Prometheus Integration
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'nvidia_gpu'
    static_configs:
      - targets: ['localhost:9400']
    metrics_path: '/metrics'
    scheme: 'http'
```

### 2. Grafana Dashboard
```json
{
  "dashboard": {
    "panels": [
      {
        "title": "GPU Utilization",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "DCGM_FI_DEV_GPU_UTIL",
            "legendFormat": "GPU {{gpu}}"
          }
        ]
      }
    ]
  }
}
```

## Best Practices

### 1. Monitoring Strategy
- Implement multi-level monitoring (system, node, job)
- Set up automated alerting for critical metrics
- Maintain historical data for trend analysis
- Regular health checks and proactive maintenance

### 2. Resource Management
- Define clear GPU allocation policies
- Monitor resource utilization patterns
- Implement fair-share scheduling
- Regular capacity planning reviews

### 3. Performance Optimization
- Track and optimize PCIe performance
- Monitor memory usage patterns
- Regular driver and firmware updates
- Performance baseline monitoring

### 4. Security Considerations
- Role-based access control
- Secure monitoring data transmission
- Regular security audits
- Compliance monitoring

## Troubleshooting Guide

### Common Issues

1. Resource Contention
   - Symptom: Unexpectedly low GPU utilization
   - Check: Process conflicts, memory leaks
   - Solution: Implement proper resource isolation

2. Performance Degradation
   - Symptom: Slower than expected processing
   - Check: PCIe configuration, thermal throttling
   - Solution: Verify hardware configuration, improve cooling

3. Monitoring System Issues
   - Symptom: Missing or incomplete data
   - Check: Collector services, network connectivity
   - Solution: Verify service status, network paths

## Related Documentation
- [[Tools - nvidia-smi Consolidated]] - NVIDIA SMI guide
- [[Tools - DGCM]] - DCGM reference
- [[SysAdmin - NVIDIA Update Script]] - Driver management
- [[Troubleshooting Decision Tree]] - Systematic problem resolution

## Tags
#gpu #monitoring #tools #management #hpc #performance