# mprime: Comprehensive Guide and Usage

## What is mprime?

mprime is the Linux version of Prime95, a popular program used for stress testing CPUs and memory subsystems. It's particularly known for:

1. Finding Mersenne prime numbers
2. Stress testing computer hardware
3. Benchmarking CPU and memory performance

mprime is highly regarded in the HPC community for its ability to push systems to their limits, making it excellent for stability testing and performance validation.

## Installation

mprime is not typically available in standard repositories. To install:

1. Download from the official GIMPS (Great Internet Mersenne Prime Search) website:
   ```
   wget https://www.mersenne.org/download/software/v30/mprime-v30.7.2.linux64.tar.gz
   ```

2. Extract the archive:
   ```
   tar -xvzf mprime-v30.7.2.linux64.tar.gz
   ```

3. Move to a suitable location:
   ```
   sudo mv mprime /usr/local/bin/
   ```

## Basic Usage

To start mprime in interactive mode:
```
mprime
```

You'll be presented with a menu:

1. Just stress testing
2. Timed stress testing
3. Work on actual Mersenne prime problems

For most HPC purposes, options 1 or 2 are typically used.

## Advanced Usage

### Command-Line Options

mprime can be run with various command-line options:

- `-t`: Run torture test
- `-m`: Run Mersenne prime test
- `-a`: Run automatic self-test

Example:
```
mprime -t
```

### Configuration File

mprime uses a configuration file named `local.txt`. You can edit this to customize the testing parameters:

```
CPUSupportsAVX=1
MinTortureFFT=8
MaxTortureFFT=4096
TortureTime=3
UsePrimenet=0
```

### Specific Test Sizes

To run tests with specific FFT sizes:
```
mprime -t -m8 -m4096
```
This runs tests using FFT sizes from 8K to 4096K.

## HPC-Specific Usage

In HPC environments, mprime is particularly useful for:

1. System Stability Testing
   Run long-duration tests to ensure system stability:
   ```
   mprime -t -t24  # Run torture test for 24 hours
   ```

2. CPU and Memory Stress Testing
   Create a script to stress test specific cores:
   ```bash
   #!/bin/bash
   # stress_test.sh
   cores=$1
   duration=$2
   
   for i in $(seq 0 $((cores-1))); do
     taskset -c $i mprime -t -t$duration &
   done
   wait
   ```
   Usage: `./stress_test.sh 64 4` (test 64 cores for 4 hours)

3. Performance Benchmarking
   Use mprime's built-in benchmarking:
   ```
   mprime -b
   ```

4. Thermal Testing
   Monitor temperatures while running mprime:
   ```bash
   #!/bin/bash
   mprime -t &
   while true; do
     sensors | grep "Core"
     sleep 5
   done
   ```

5. Power Consumption Analysis
   Run mprime while monitoring power draw:
   ```bash
   mprime -t &
   while true; do
     ipmitool sensor reading "PSU1 Input Power"
     sleep 5
   done
   ```

## Best Practices for HPC Use

1. Baseline Testing: Establish performance baselines for your systems using mprime.

2. Pre-deployment Validation: Run mprime tests on new hardware before deploying to production.

3. Regular Health Checks: Schedule periodic mprime runs to ensure ongoing system stability.

4. Thermal Management: Use mprime in conjunction with temperature monitoring to validate cooling systems.

5. Custom Test Profiles: Develop custom mprime test profiles that match your typical workload characteristics.

6. Parallel Testing: For large clusters, run mprime across multiple nodes simultaneously to stress test the entire system.

7. Result Logging: Implement comprehensive logging of mprime results for trend analysis.

## Interpreting Results

- Errors or crashes during tests indicate hardware instability.
- Compare benchmark results across similar systems to identify underperforming hardware.
- Monitor for thermal throttling during extended runs.

## Cautions and Considerations

1. High Load: mprime generates extremely high CPU loads. Ensure adequate cooling.
2. Power Consumption: Be aware of increased power draw during tests.
3. Production Impact: Avoid running on production systems without proper planning.
4. Complementary Testing: Use mprime alongside other testing tools for comprehensive validation.

Remember, while mprime is powerful for stress testing and benchmarking, it should be used judiciously, especially in production HPC environments. Always have a clear testing plan and monitor systems closely during mprime runs.