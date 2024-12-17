# System Stress Testing with stress-ng

## Overview
stress-ng is a powerful tool for system stress testing and performance validation. It can stress various system resources including CPU, memory, I/O, and more.

## Installation
```bash
# Install stress-ng
sudo apt install stress-ng    # Ubuntu/Debian
sudo dnf install stress-ng    # RHEL/Rocky
```

## Basic Usage

### CPU Stress Testing
```bash
# Stress all CPU cores
stress-ng --cpu $(nproc) --timeout 60s

# Stress with specific CPU methods
stress-ng --cpu 4 --cpu-method all --timeout 300s

# CPU cache testing
stress-ng --cache 4 --timeout 60s
```

### Memory Testing
```bash
# Stress test memory
stress-ng --vm 4 --vm-bytes 80% --timeout 60s

# Memory with specific pattern
stress-ng --vm 2 --vm-bytes 1G --vm-method binary --timeout 60s
```

### I/O Testing
```bash
# Disk I/O stress
stress-ng --io 4 --timeout 60s

# Specific I/O operations
stress-ng --hdd 2 --hdd-bytes 2G --timeout 60s
```

## Advanced Testing Scripts

### Comprehensive System Test
```bash
#!/bin/bash
# Full system stress test

run_stress_test() {
    local duration=$1
    local report_file="stress_test_$(date +%Y%m%d_%H%M%S).log"
    
    echo "Starting stress test - Duration: ${duration}s" | tee -a "$report_file"
    
    stress-ng \
        --cpu $(nproc) \
        --vm 2 \
        --vm-bytes 80% \
        --io 4 \
        --timeout "${duration}s" \
        --metrics-brief | tee -a "$report_file"
        
    echo "Test completed - $(date)" | tee -a "$report_file"
}
```

### Temperature Monitoring During Stress
```bash
#!/bin/bash
# Monitor temperatures during stress test

monitor_stress() {
    local duration=$1
    local interval=5
    local log_file="temp_log_$(date +%Y%m%d_%H%M%S).csv"
    
    echo "Timestamp,CPU_Temp,GPU_Temp" > "$log_file"
    
    stress-ng --cpu $(nproc) --timeout "${duration}s" &
    
    while ps aux | grep -q "[s]tress-ng"; do
        cpu_temp=$(sensors | grep "Package id 0" | awk '{print $4}' | tr -d '+°C')
        gpu_temp=$(nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader)
        echo "$(date +%s),$cpu_temp,$gpu_temp" >> "$log_file"
        sleep $interval
    done
}
```

## Specific Test Scenarios

### Memory Leak Detection
```bash
#!/bin/bash
# Test for memory leaks

test_memory_leaks() {
    stress-ng --vm 1 \
        --vm-bytes 75% \
        --page-in \
        --metrics-brief \
        --timeout 300s \
        --verify
}
```

### I/O Performance Baseline
```bash
#!/bin/bash
# Establish I/O performance baseline

test_io_performance() {
    local test_file="/tmp/stress_io_test"
    
    stress-ng --iomix 4 \
        --hdd 2 \
        --hdd-bytes 1G \
        --temp-path "$test_file" \
        --metrics-brief \
        --timeout 60s
        
    rm -f "$test_file"
}
```

## Integration with Monitoring

### Prometheus Metrics
```bash
#!/bin/bash
# Generate stress test metrics for Prometheus

generate_stress_metrics() {
    echo "# HELP stress_test_cpu_load CPU load during stress test"
    echo "# TYPE stress_test_cpu_load gauge"
    
    cpu_load=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}')
    echo "stress_test_cpu_load $cpu_load"
    
    echo "# HELP stress_test_memory_used Memory used during stress test"
    echo "# TYPE stress_test_memory_used gauge"
    
    memory_used=$(free | grep Mem | awk '{print $3}')
    echo "stress_test_memory_used $memory_used"
}
```

### Grafana Dashboard Configuration
```json
{
  "panels": [
    {
      "title": "CPU Load During Stress",
      "type": "graph",
      "targets": [
        {
          "expr": "stress_test_cpu_load",
          "legendFormat": "CPU Load"
        }
      ]
    },
    {
      "title": "Memory Usage During Stress",
      "type": "graph",
      "targets": [
        {
          "expr": "stress_test_memory_used",
          "legendFormat": "Memory Used"
        }
      ]
    }
  ]
}
```

## Common Test Patterns

### System Validation Test Suite
```bash
#!/bin/bash
# Comprehensive validation suite

run_validation_suite() {
    local log_dir="validation_$(date +%Y%m%d_%H%M%S)"
    mkdir -p "$log_dir"
    
    # CPU Test
    stress-ng --cpu $(nproc) --cpu-method all \
        --metrics-brief --timeout 300s \
        > "$log_dir/cpu_test.log" 2>&1
    
    # Memory Test
    stress-ng --vm 4 --vm-bytes 80% \
        --metrics-brief --timeout 300s \
        > "$log_dir/memory_test.log" 2>&1
    
    # I/O Test
    stress-ng --io 4 --hdd 2 --hdd-bytes 1G \
        --metrics-brief --timeout 300s \
        > "$log_dir/io_test.log" 2>&1
    
    # Network Test
    stress-ng --sock 4 --sock-ops 1000 \
        --metrics-brief --timeout 300s \
        > "$log_dir/network_test.log" 2>&1
}
```

## Tags
#testing #performance #stress-testing #system-validation