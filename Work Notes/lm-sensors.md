# lm-sensors: Comprehensive Guide and Usage

## What is lm-sensors?

lm-sensors (Linux monitoring sensors) is a free, open-source tool for Linux that provides utilities to monitor hardware sensors, including CPU temperatures, fan speeds, and voltage readings. It's particularly useful in HPC environments for monitoring system health and preventing hardware failures due to overheating or other issues.

## Key Components

1. libsensors: The core library
2. sensors: Command-line tool to display readings
3. sensord: Daemon to log sensor readings
4. fancontrol: Script to control fan speeds (if supported by hardware)

## Installation

### On Ubuntu/Debian:
```bash
sudo apt update
sudo apt install lm-sensors
```

### On CentOS/RHEL:
```bash
sudo yum install lm_sensors
```

## Basic Setup and Usage

1. Detect sensors:
   ```bash
   sudo sensors-detect
   ```
   Follow the prompts, answering 'YES' to relevant questions.

2. View sensor readings:
   ```bash
   sensors
   ```

## Advanced Usage

### 1. Continuous Monitoring
To watch sensor readings in real-time:
```bash
watch sensors
```

### 2. Specific Chip Monitoring
To monitor a specific chip:
```bash
sensors chip_name
```
Replace `chip_name` with the actual chip name (e.g., coretemp-isa-0000).

### 3. Custom Output Format
For scripting, you can use custom output:
```bash
sensors -u
```
This provides machine-readable output.

### 4. Fan Control
If your hardware supports it:
```bash
sudo pwmconfig
```
This sets up fan control. Then start the fan control service:
```bash
sudo systemctl start fancontrol
```

## HPC-Specific Usage

In HPC environments, lm-sensors is particularly useful for:

1. Node Health Monitoring
   Create a script to regularly check temperatures:
   ```bash
   #!/bin/bash
   max_temp=$(sensors | grep 'Core' | awk '{print $3}' | tr -d '+°C' | sort -rn | head -n1)
   if (( $(echo "$max_temp > 80" | bc -l) )); then
       echo "High temperature detected: $max_temp°C"
   fi
   ```

2. Cluster-wide Monitoring
   Use with tools like Ganglia or Prometheus to collect data across all nodes.

3. Job Scheduler Integration
   Integrate temperature checks into your job scheduler to avoid scheduling jobs on overheating nodes.

4. Power Usage Estimation
   Some sensors can provide power usage data, useful for energy efficiency monitoring.

5. GPU Temperature Monitoring
   For systems with NVIDIA GPUs, combine with `nvidia-smi`:
   ```bash
   nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader
   ```

## Best Practices for HPC Use

1. Baseline Establishment: Determine normal temperature ranges for your hardware under various load conditions.

2. Regular Monitoring: Set up automated, regular checks of sensor data.

3. Alerting System: Implement alerts for when temperatures or other readings exceed predefined thresholds.

4. Correlation with Workloads: Monitor how different types of HPC workloads affect temperature and other sensor readings.

5. Seasonal Adjustments: Be aware of how ambient temperature changes (e.g., summer vs. winter) affect your readings and adjust your baselines accordingly.

6. Maintenance Scheduling: Use sensor data to inform preventive maintenance schedules.

## Troubleshooting Common Issues

1. Missing Sensors: Some hardware may not be detected automatically. Check your BIOS settings and kernel modules.

2. Inaccurate Readings: Calibrate your sensors if you suspect inaccuracies.

3. High Temperatures: Check for dust buildup, proper airflow, and consider if thermal paste needs reapplication.

4. Inconsistent Fan Speeds: This might indicate failing fans or issues with fan control settings.

## Integration with Other Tools

1. Grafana: Visualize sensor data over time.
2. Nagios/Icinga: Set up monitoring and alerts based on sensor data.
3. Ansible: Use for cluster-wide sensor checks and configuration.

Remember, while lm-sensors is powerful for hardware monitoring, it should be used alongside other monitoring tools for comprehensive system oversight in HPC environments.


# Using lm-sensors for Quick Heat-Related Troubleshooting in HPC

## Quick Check Commands

1. Basic Temperature Overview:
   ```bash
   sensors | grep -E 'Core|Package'
   ```
   This command quickly shows CPU core and package temperatures.

2. Find Maximum CPU Temperature:
   ```bash
   sensors | grep 'Core' | awk '{print $3}' | tr -d '+°C' | sort -rn | head -n1
   ```
   This extracts and displays the highest CPU core temperature.

3. Check for Critical Temperatures:
   ```bash
   sensors | grep -E 'Core|Package' | awk '$3 > 80 {print $0}'
   ```
   This highlights any CPU temperatures over 80°C (adjust the threshold as needed).

## Quick Troubleshooting Script

Create a script named `check_temp.sh`:

```bash
#!/bin/bash

# Function to get max temperature
get_max_temp() {
    sensors | grep 'Core' | awk '{print $3}' | tr -d '+°C' | sort -rn | head -n1
}

# Function to get average temperature
get_avg_temp() {
    sensors | grep 'Core' | awk '{sum+=$3; count++} END {print sum/count}' | tr -d '+°C'
}

# Main check
max_temp=$(get_max_temp)
avg_temp=$(get_avg_temp)

echo "Maximum CPU Temperature: ${max_temp}°C"
echo "Average CPU Temperature: ${avg_temp}°C"

if (( $(echo "$max_temp > 80" | bc -l) )); then
    echo "WARNING: High temperature detected!"
    echo "Checking for potential causes..."
    
    # Check CPU usage
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
    echo "Current CPU Usage: ${cpu_usage}%"
    
    # Check for high-load processes
    echo "Top CPU-consuming processes:"
    ps aux --sort=-%cpu | head -n 5
    
    # Check fan speeds
    echo "Fan Speeds:"
    sensors | grep -i fan
    
    # Check for recent temperature trend
    echo "Recent temperature trend (last 5 minutes):"
    sar -t -f /var/log/sa/sa$(date +%d) | tail -n 6
else
    echo "Temperatures are within normal range."
fi
```

Make the script executable:
```bash
chmod +x check_temp.sh
```

Run it for a quick heat-related diagnostic:
```bash
./check_temp.sh
```

## Interpreting Results

- Max Temperature > 80°C: Indicates potential overheating. Investigate immediately.
- Max Temperature 70-80°C: High but may be normal under heavy load. Monitor closely.
- Max Temperature < 70°C: Generally considered safe for most CPUs.

## Quick Remediation Steps

If high temperatures are detected:

1. Check system load: High temperatures with low CPU usage might indicate cooling issues.
2. Verify fan operation: Ensure all fans are running at appropriate speeds.
3. Check ambient temperature: Unusually high room temperature can affect system cooling.
4. Inspect for airflow blockages: Ensure proper ventilation around the system.
5. Consider throttling or migrating high-load jobs if temperatures remain critical.

## Integration with HPC Environment

1. Run this script as part of your job scheduler's prologue and epilogue scripts.
2. Implement in cluster monitoring systems for continuous temperature oversight.
3. Use in conjunction with GPU temperature monitoring for systems with GPUs.

Remember, while this provides a quick check, persistent or severe temperature issues should prompt a thorough investigation, possibly including physical inspection of the cooling systems.


# Gathering Long-Term Temperature Trends in HPC Environments

## 1. Data Collection

### Option A: Using lm-sensors with a Custom Script

1. Create a script named `log_temperatures.sh`:

```bash
#!/bin/bash

LOG_FILE="/var/log/temperatures.log"
INTERVAL=300  # 5 minutes

while true; do
    timestamp=$(date +"%Y-%m-%d %H:%M:%S")
    temps=$(sensors | grep -E 'Core|Package' | awk '{print $3}' | tr -d '+°C')
    
    echo "$timestamp $(hostname) $temps" >> $LOG_FILE
    sleep $INTERVAL
done
```

2. Make the script executable:
```bash
chmod +x log_temperatures.sh
```

3. Run as a service:
Create a systemd service file `/etc/systemd/system/temp-logger.service`:

```
[Unit]
Description=Temperature Logging Service
After=network.target

[Service]
ExecStart=/path/to/log_temperatures.sh
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

Enable and start the service:
```bash
sudo systemctl enable temp-logger.service
sudo systemctl start temp-logger.service
```

### Option B: Using Collectd

1. Install collectd:
```bash
sudo apt install collectd
```

2. Configure collectd to collect temperature data. Edit `/etc/collectd/collectd.conf`:

```
LoadPlugin sensors
<Plugin sensors>
    SensorConfigFile "/etc/sensors3.conf"
    Sensor "coretemp-isa-0000/temperature-input"
    IgnoreSelected false
</Plugin>
```

3. Restart collectd:
```bash
sudo systemctl restart collectd
```

## 2. Data Storage

### Option A: Simple File-based Storage

- Use the log file created by the custom script.

### Option B: Time-Series Database

1. Install InfluxDB:
```bash
sudo apt install influxdb
sudo systemctl enable influxdb
sudo systemctl start influxdb
```

2. Create a database:
```bash
influx -execute 'CREATE DATABASE temperature_trends'
```

3. If using collectd, configure it to write to InfluxDB. Add to collectd.conf:

```
LoadPlugin network
<Plugin network>
    Server "127.0.0.1" "25826"
</Plugin>
```

4. Configure InfluxDB to accept collectd data. Edit `/etc/influxdb/influxdb.conf`:

```
[[collectd]]
  enabled = true
  bind-address = ":25826"
  database = "temperature_trends"
  retention-policy = ""
```

## 3. Data Analysis and Visualization

### Option A: Simple Analysis with Command-line Tools

1. Calculate daily average temperatures:
```bash
awk '{sum+=$4; count++} END {print "Average temp:", sum/count}' /var/log/temperatures.log
```

2. Find maximum temperature:
```bash
sort -k4 -nr /var/log/temperatures.log | head -n1
```

### Option B: Using Grafana for Visualization

1. Install Grafana:
```bash
sudo apt install grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

2. Configure Grafana to use InfluxDB as a data source.

3. Create dashboards to visualize temperature trends over time.

## 4. Automated Analysis and Alerting

1. Create a script to analyze trends and send alerts:

```python
import pandas as pd
from influxdb import InfluxDBClient

client = InfluxDBClient(host='localhost', port=8086, database='temperature_trends')

# Query last 30 days of data
query = 'SELECT mean("value") FROM "temperature" WHERE time > now() - 30d GROUP BY time(1d)'
result = client.query(query)

# Convert to pandas DataFrame
df = pd.DataFrame(result.get_points())
df['time'] = pd.to_datetime(df['time'])
df.set_index('time', inplace=True)

# Calculate trend
trend = df['mean'].diff().mean()

if trend > 0.1:  # If average daily increase is more than 0.1°C
    print("Alert: Increasing temperature trend detected!")
    # Add code here to send an alert (e.g., email, Slack notification)
```

2. Schedule this script to run regularly using cron.

## 5. Best Practices for Long-Term Trend Analysis

1. Consistent Data Collection: Ensure data is collected at regular intervals.
2. Data Normalization: Account for factors like ambient temperature and workload.
3. Multiple Metrics: Collect related metrics (e.g., fan speeds, power consumption) for comprehensive analysis.
4. Regular Review: Set up periodic reviews of long-term trends (e.g., monthly, quarterly).
5. Correlation with Events: Note system changes, maintenance activities, or environmental changes that might affect temperatures.
6. Seasonal Adjustments: Account for seasonal temperature variations in your analysis.

## 6. Using Trends for Proactive Management

1. Capacity Planning: Use trends to predict future cooling needs.
2. Maintenance Scheduling: Schedule preventive maintenance based on temperature trends.
3. Efficiency Optimization: Identify opportunities to improve cooling efficiency.
4. Hardware Lifecycle Management: Use trends to inform decisions about hardware upgrades or replacements.

Remember, long-term temperature trend analysis is most effective when integrated into your overall HPC management strategy, informing decisions about workload distribution, infrastructure upgrades, and operational practices.