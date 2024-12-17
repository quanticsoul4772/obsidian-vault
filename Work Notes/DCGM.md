# NVIDIA Data Center GPU Manager (DCGM): Installation and Usage Guide

## What is NVIDIA DCGM?

NVIDIA Data Center GPU Manager (DCGM) is a suite of tools for managing and monitoring NVIDIA GPUs in cluster and data center environments. It provides advanced monitoring, management, and health diagnostics for NVIDIA GPUs, making it particularly useful in HPC environments.

Key features:
- Health and diagnostics monitoring
- Process statistics tracking
- Active health checks and failure prediction
- GPU telemetry collection and export
- GPU configuration management

## Installation

### Prerequisites
- NVIDIA GPU drivers (version 418.40+)
- CUDA Toolkit (optional, but recommended)

### Installation Steps

1. For Ubuntu/Debian:
   ```bash
   sudo apt-get update
   sudo apt-get install datacenter-gpu-manager
   ```

2. For RHEL/CentOS:
   ```bash
   sudo yum install epel-release
   sudo yum install datacenter-gpu-manager
   ```

3. Verify installation:
   ```bash
   dcgmi discovery -l
   ```

## Basic Usage

1. Start the DCGM service:
   ```bash
   sudo systemctl start nvidia-dcgm
   ```

2. Enable DCGM to start at boot:
   ```bash
   sudo systemctl enable nvidia-dcgm
   ```

3. Check DCGM status:
   ```bash
   sudo systemctl status nvidia-dcgm
   ```

## DCGM Commands and Tools

### dcgmi (DCGM Command Line Interface)

1. List all GPUs:
   ```bash
   dcgmi discovery -l
   ```

2. Get GPU info:
   ```bash
   dcgmi dmon
   ```

3. Run diagnostics:
   ```bash
   dcgmi diag -r 1
   ```

### DCGM API

DCGM provides APIs for C, Python, and Go for custom integration:

Python example:
```python
import pydcgm
dcgm_handle = pydcgm.DcgmHandle()
dcgm_system = dcgm_handle.GetSystem()
gpus = dcgm_system.discovery.GetAllGpuIds()
for gpu in gpus:
    print(dcgm_system.discovery.GetGpuAttributes(gpu))
```

### DCGM Exporter

DCGM Exporter allows exporting GPU metrics to monitoring systems like Prometheus:

1. Run DCGM Exporter:
   ```bash
   dcgm-exporter
   ```

2. Access metrics:
   ```bash
   curl localhost:9400/metrics
   ```

## Advanced Usage in HPC Environments

1. Health Checks:
   ```bash
   dcgmi health -g 0 -c -j
   ```
   This runs a health check on GPU 0 and outputs in JSON format.

2. GPU Grouping:
   ```bash
   dcgmi group -c mygroup
   dcgmi group -g mygroup -a 0,1
   ```
   This creates a group named "mygroup" and adds GPUs 0 and 1 to it.

3. Policy Management:
   ```bash
   dcgmi policy -g mygroup --set 0,0,0
   ```
   This sets a policy for the "mygroup" group.

4. Profiling:
   ```bash
   dcgmi profile -g mygroup --pause
   # Run your application
   dcgmi profile -g mygroup --resume
   ```
   This allows profiling of GPU usage for specific applications.

## Integration with Job Schedulers

1. Slurm Integration:
   Add to slurm.conf:
   ```
   GresTypes=gpu
   NodeName=node[1-10] Gres=gpu:4
   ```

2. Use DCGM for GPU selection:
   In Slurm's gres.conf:
   ```
   NodeName=node[1-10] Name=gpu File=/dev/nvidia[0-3]
   ```

## Best Practices

1. Regular Health Checks: Schedule regular GPU health checks using DCGM.
2. Monitoring Integration: Use DCGM Exporter with Prometheus and Grafana for comprehensive monitoring.
3. Automated Remediation: Use DCGM's health check results to trigger automated maintenance tasks.
4. Performance Optimization: Use DCGM's profiling capabilities to optimize GPU usage in your applications.
5. Capacity Planning: Leverage DCGM's long-term metrics collection for informed capacity planning.

## Troubleshooting

1. If DCGM service fails to start, check GPU driver compatibility.
2. For "GPU not supported" errors, ensure your GPU model is supported by DCGM.
3. Use `dcgmi logs` to view DCGM logs for troubleshooting.

Remember, DCGM is a powerful tool that complements nvidia-smi and other GPU management utilities. Its integration capabilities make it particularly valuable in large-scale HPC environments.