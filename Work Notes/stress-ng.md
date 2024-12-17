# stress-ng: Comprehensive Guide and Usage

## What is stress-ng?

stress-ng is a powerful, flexible tool designed to impose varying levels of stress on computer systems. It's particularly useful for stress testing hardware, benchmarking, and identifying system stability issues under high load conditions.

## Key Features:

1. Supports a wide range of stress tests for various system components
2. Highly configurable with numerous options
3. Can stress test CPU, memory, I/O, and more
4. Provides detailed metrics and statistics
5. Useful for both quick checks and extended stress testing

## Installation

### On Ubuntu/Debian-based systems:
```
sudo apt update
sudo apt install stress-ng
```

### On Rocky Linux/RHEL-based systems:
```
sudo dnf install epel-release
sudo dnf install stress-ng
```

## Basic Usage

The general syntax for stress-ng is:
```
stress-ng [options] [stressors]
```

## Common Use Cases and Examples

1. CPU Stress Test:
   ```
   stress-ng --cpu 4 --timeout 60s
   ```
   This runs 4 CPU-intensive tasks for 60 seconds.

2. Memory Stress Test:
   ```
   stress-ng --vm 2 --vm-bytes 1G --timeout 30s
   ```
   This creates 2 workers, each allocating and freeing 1GB of memory for 30 seconds.

3. I/O Stress Test:
   ```
   stress-ng --io 4 --timeout 60s
   ```
   This runs 4 I/O-intensive tasks for 60 seconds.

4. Combined Stress Test:
   ```
   stress-ng --cpu 2 --io 2 --vm 2 --vm-bytes 128M --timeout 60s
   ```
   This combines CPU, I/O, and memory stress tests.

5. Specific CPU Instruction Set Test:
   ```
   stress-ng --cpu 1 --cpu-method ackermann --timeout 30s
   ```
   This tests the CPU using the Ackermann function.

6. Filesystem Stress:
   ```
   stress-ng --hdd 2 --hdd-bytes 1G --timeout 60s
   ```
   This writes and reads 1GB of data using 2 workers.

## Advanced Usage

1. Stress Test with Performance Metrics:
   ```
   stress-ng --cpu 4 --metrics-brief --timeout 60s
   ```
   This provides a summary of performance metrics after the test.

2. Randomized Stressors:
   ```
   stress-ng --random 8 --timeout 5m
   ```
   This runs 8 random stressors for 5 minutes.

3. Thermal Stress Test:
   ```
   stress-ng --cpu 4 --cpu-method all --tz --timeout 10m
   ```
   This runs all CPU methods and monitors thermal zones for 10 minutes.

4. Network Stress Test:
   ```
   stress-ng --sock 4 --timeout 1m
   ```
   This creates 4 workers that perform various socket operations.

## Best Practices for HPC Environments

1. Start with lower stress levels and gradually increase.
2. Monitor system temperatures during stress tests.
3. Use `--metrics-brief` or `--yaml` for detailed performance data.
4. Combine different stressors to simulate real-world HPC workloads.
5. For cluster testing, consider running stress-ng across multiple nodes simultaneously.

## Interpreting Results

- Look for system crashes, freezes, or unexpected reboots during tests.
- Check system logs for any errors or warnings.
- Compare performance metrics across different hardware configurations.
- Use thermal monitoring to identify potential cooling issues.

## Cautions

- Stress testing can push hardware to its limits. Ensure adequate cooling.
- On production systems, start with less intensive tests to avoid disruptions.
- Some stress tests (like I/O) can cause wear on SSDs if run excessively.

Remember, while stress-ng is a powerful tool for system validation, it should be part of a broader testing strategy in HPC environments.

# Creating Custom Stress Tests for HPC Workloads with stress-ng

## Understanding Your HPC Workload

Before creating a custom stress test, analyze your HPC workload:

1. CPU usage patterns (floating-point vs. integer operations)
2. Memory access patterns and volume
3. I/O characteristics (file sizes, read/write ratios)
4. Network usage
5. Specific libraries or instruction sets used

## Building Custom Stress Tests

### 1. CPU-Intensive Workloads

For workloads heavy on computation:

```bash
stress-ng --cpu 8 --cpu-method matrixprod --cpu-ops 1000000 --timeout 10m
```

This runs 8 CPU stressors using matrix multiplication for 10 minutes.

For more specific CPU operations:

```bash
stress-ng --cpu 4 --cpu-method all --cpu-ops 1000000 --timeout 15m
```

This cycles through all CPU stressors, useful for mixed workloads.

### 2. Memory-Intensive Workloads

For applications with high memory usage:

```bash
stress-ng --vm 4 --vm-bytes 4G --vm-method all --timeout 10m
```

This allocates 4GB per worker, cycling through different memory stress methods.

### 3. I/O-Intensive Workloads

For workloads with heavy disk I/O:

```bash
stress-ng --iomix 2 --iomix-bytes 5G --timeout 15m
```

This performs a mix of I/O operations, writing and reading 5GB per worker.

### 4. Network-Intensive Workloads

For distributed computing workloads:

```bash
stress-ng --sock 4 --sock-ops 1000000 --timeout 10m
```

This stresses the network stack with socket operations.

### 5. Combined Workload Simulation

To simulate a mixed HPC workload:

```bash
stress-ng --cpu 4 --cpu-method matrixprod \
          --vm 2 --vm-bytes 2G \
          --iomix 1 --iomix-bytes 2G \
          --sock 2 \
          --timeout 20m
```

This combines CPU, memory, I/O, and network stressors.

## Tailoring for Specific HPC Applications

### Example: CFD Simulation Stress Test

Computational Fluid Dynamics often involves heavy floating-point operations and large memory usage:

```bash
stress-ng --cpu 8 --cpu-method float64 --vm 4 --vm-bytes 4G --timeout 30m
```

### Example: Genomic Sequencing Stress Test

Genomic sequencing often involves string operations and high I/O:

```bash
stress-ng --cpu 6 --cpu-method strcmp --iomix 2 --iomix-bytes 10G --timeout 25m
```

## Advanced Customization

### 1. Using Multiple Stressors

```bash
stress-ng --cpu 4 --cpu-method matrixprod \
          --cpu 4 --cpu-method float64 \
          --timeout 15m
```

This runs two different CPU stressors simultaneously.

### 2. Stressor Affinities

For NUMA systems:

```bash
stress-ng --cpu 0-3 --cpu-method matrixprod \
          --cpu 4-7 --cpu-method float64 \
          --timeout 15m
```

This pins different stressors to specific CPU cores.

### 3. Monitoring and Metrics

Add `--metrics-brief` for a summary or `--yaml` for detailed metrics:

```bash
stress-ng --cpu 8 --cpu-method matrixprod --timeout 10m --metrics-brief
```

## Best Practices

1. Start with lower intensity and gradually increase.
2. Run tests for extended periods to catch time-dependent issues.
3. Combine stress-ng with system monitoring tools (e.g., sar, iostat, mpstat).
4. Test on a representative, non-production system first.
5. Document your custom stress tests for reproducibility.

## Automating Custom Tests

Create a shell script to run a series of custom tests:

```bash
#!/bin/bash

# CFD-like workload
stress-ng --cpu 8 --cpu-method float64 --vm 4 --vm-bytes 4G --timeout 15m

# Genomic sequencing-like workload
stress-ng --cpu 6 --cpu-method strcmp --iomix 2 --iomix-bytes 10G --timeout 15m

# Add more custom tests as needed
```

Remember to adjust the parameters based on your specific hardware and workload characteristics.

# Analyzing Stress Test Results and Integrating into HPC Validation Framework

## Part 1: Analyzing Stress Test Results

### 1. Collecting Data

- Use stress-ng's built-in metrics: `--metrics-brief` or `--yaml` for detailed output
- Employ system monitoring tools alongside stress-ng:
  - `sar` for CPU, memory, and I/O statistics
  - `iostat` for detailed I/O metrics
  - `mpstat` for per-processor statistics
  - `nvidia-smi` for GPU metrics (if applicable)

### 2. Key Metrics to Analyze

- CPU utilization and load average
- Memory usage and swap activity
- I/O operations per second (IOPS) and throughput
- Network throughput and latency
- Temperature and power consumption
- Error rates or system messages

### 3. Analysis Techniques

a. Baseline Comparison
   - Compare results against known-good baseline performance
   - Look for significant deviations in key metrics

b. Trend Analysis
   - Plot metrics over time to identify patterns or degradation
   - Use tools like Grafana or Matplotlib for visualization

c. Statistical Analysis
   - Calculate mean, median, and standard deviation of performance metrics
   - Identify outliers or unexpected variations

d. Performance Ratio Analysis
   - Compare actual vs. expected performance ratios
   - Example: GFLOPS achieved vs. theoretical peak performance

### 4. Interpreting Results

- Look for:
  - Unexpected performance drops
  - Thermal throttling
  - Error messages in system logs
  - Inconsistent performance across similar nodes
- Consider the impact of results on real-world HPC workloads

## Part 2: Integrating into HPC Validation Framework

### 1. Establish a Validation Protocol

a. Define validation stages:
   - Initial hardware validation
   - Pre-production testing
   - Regular health checks
   - Post-maintenance validation

b. Set acceptance criteria for each stage
   - Example: CPU must sustain 95% of rated FLOPS for 24 hours

### 2. Automate Test Execution

a. Create scripts to run custom stress tests
   ```bash
   #!/bin/bash
   # Run CPU stress test
   stress-ng --cpu 8 --cpu-method matrixprod --timeout 1h --metrics-brief > cpu_test_results.txt
   
   # Run memory stress test
   stress-ng --vm 4 --vm-bytes 4G --timeout 1h --metrics-brief > memory_test_results.txt
   
   # Add more tests as needed
   ```

b. Use job schedulers (e.g., Slurm, PBS) to distribute tests across the cluster
   ```bash
   srun -N 10 ./run_stress_tests.sh
   ```

### 3. Implement Result Collection and Storage

a. Centralized logging system (e.g., ELK stack, Splunk)
b. Time-series database for metrics (e.g., InfluxDB, Prometheus)
c. Store raw test outputs in a structured format (JSON, CSV)

### 4. Develop Analysis Pipeline

a. Automate result parsing and basic analysis
   ```python
   import pandas as pd
   import matplotlib.pyplot as plt
   
   # Load and process data
   data = pd.read_csv('stress_test_results.csv')
   
   # Perform analysis
   avg_performance = data['performance_metric'].mean()
   
   # Visualize results
   plt.plot(data['time'], data['performance_metric'])
   plt.savefig('performance_over_time.png')
   ```

b. Implement alerting for failed tests or anomalies

### 5. Integration with Existing Systems

a. Link with asset management system
b. Integrate with change management processes
c. Feed results into capacity planning tools

### 6. Continuous Improvement

a. Regularly review and update test suite
b. Adjust acceptance criteria based on evolving requirements
c. Incorporate lessons learned from production issues

### 7. Reporting and Dashboards

a. Create summary reports for stakeholders
b. Develop real-time dashboards for system health
c. Maintain a historical performance database

### 8. Validation Workflow Example

1. New hardware arrival → Run full stress test suite
2. Passing nodes → Move to pre-production testing
3. Regular intervals (e.g., weekly) → Run subset of tests
4. After maintenance → Run relevant component tests
5. Periodic (e.g., quarterly) → Full system validation

### 9. Training and Documentation

a. Train staff on interpreting test results
b. Document procedures for running tests and handling failures
c. Maintain a knowledge base of common issues and resolutions

Remember, the key is to create a systematic, repeatable process that provides confidence in your HPC system's capabilities and helps identify potential issues before they impact production workloads.