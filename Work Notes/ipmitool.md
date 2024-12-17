# ipmitool: Comprehensive Guide and Advanced Usage

## 1. Introduction to ipmitool

ipmitool is a powerful utility for managing and configuring IPMI-enabled devices. It provides a command-line interface for out-of-band management of server systems, allowing administrators to monitor hardware health, manage system event logs, control power, and perform various other system management tasks.

## 2. Installation

### On Ubuntu/Debian:
```bash
sudo apt update
sudo apt install ipmitool
```

### On CentOS/RHEL:
```bash
sudo yum install ipmitool
```

## 3. Basic Syntax

The general syntax for ipmitool commands is:
```
ipmitool [global options] <command> [command options]
```

Common global options include:
- `-I`: Specify the interface to use (e.g., lanplus, open)
- `-H`: Remote host IP address
- `-U`: Remote username
- `-P`: Remote password
- `-L`: Remote session privilege level

## 4. Core Functionality and Commands

### 4.1 Power Management

- Check power status:
  ```
  ipmitool power status
  ```

- Power on:
  ```
  ipmitool power on
  ```

- Power off:
  ```
  ipmitool power off
  ```

- Power cycle:
  ```
  ipmitool power cycle
  ```

- Soft shutdown:
  ```
  ipmitool power soft
  ```

### 4.2 System Information

- Get system info:
  ```
  ipmitool fru print
  ```

- Get BMC info:
  ```
  ipmitool mc info
  ```

### 4.3 Sensor Readings

- List all sensor readings:
  ```
  ipmitool sensor list
  ```

- Get specific sensor reading:
  ```
  ipmitool sensor get "CPU Temp"
  ```

### 4.4 System Event Log (SEL)

- View SEL:
  ```
  ipmitool sel list
  ```

- Clear SEL:
  ```
  ipmitool sel clear
  ```

### 4.5 Chassis Control

- Get chassis status:
  ```
  ipmitool chassis status
  ```

- Identify LED control:
  ```
  ipmitool chassis identify 30  # Turn on LED for 30 seconds
  ```

### 4.6 Boot Device Control

- Set next boot device:
  ```
  ipmitool chassis bootdev pxe  # Set to PXE boot
  ipmitool chassis bootdev disk  # Set to disk boot
  ```

## 5. Advanced Usage

### 5.1 User Management

- List users:
  ```
  ipmitool user list 1
  ```

- Set user password:
  ```
  ipmitool user set password 2 newpassword
  ```

- Add new user:
  ```
  ipmitool user set name 3 newuser
  ipmitool user set password 3 userpassword
  ipmitool user priv 3 4 1  # Set privilege level to Administrator
  ipmitool user enable 3
  ```

### 5.2 Network Configuration

- Show network settings:
  ```
  ipmitool lan print 1
  ```

- Set static IP:
  ```
  ipmitool lan set 1 ipaddr 192.168.1.100
  ipmitool lan set 1 netmask 255.255.255.0
  ipmitool lan set 1 defgw ipaddr 192.168.1.1
  ```

- Enable DHCP:
  ```
  ipmitool lan set 1 ipsrc dhcp
  ```

### 5.3 Serial Over LAN (SOL)

- Activate SOL:
  ```
  ipmitool sol activate
  ```

- Deactivate SOL:
  ```
  ipmitool sol deactivate
  ```

- Set SOL parameters:
  ```
  ipmitool sol set volatile-bit-rate 115.2  # Set baud rate
  ```

### 5.4 Firmware Management

- Get firmware version:
  ```
  ipmitool mc info
  ```

- Update firmware (vendor-specific):
  ```
  ipmitool hpm upgrade firmware.hpm
  ```

### 5.5 Raw Commands

For advanced operations or when standard commands don't suffice:
```
ipmitool raw 0x00 0x01  # Get device ID
```

### 5.6 Sensor Thresholds

- Get sensor thresholds:
  ```
  ipmitool sensor get "CPU Temp"
  ```

- Set sensor thresholds:
  ```
  ipmitool sensor thresh "CPU Temp" upper 70 75 80
  ```

### 5.7 BIOS Configuration

Some systems allow BIOS settings manipulation:
```
ipmitool chassis bootparam set bootflag force_pxe
```

## 6. HPC-Specific Usage

### 6.1 Cluster-wide Power Management

Create a script for cluster-wide power management:

```bash
#!/bin/bash
# power_manage_cluster.sh
IPMI_HOSTS="192.168.1.101 192.168.1.102 192.168.1.103"
IPMI_USER="admin"
IPMI_PASS="password"

action=$1

for host in $IPMI_HOSTS; do
    ipmitool -I lanplus -H $host -U $IPMI_USER -P $IPMI_PASS power $action
done
```

Usage: `./power_manage_cluster.sh on`

### 6.2 Temperature Monitoring

Create a script to monitor temperatures across the cluster:

```bash
#!/bin/bash
# monitor_cluster_temp.sh
IPMI_HOSTS="192.168.1.101 192.168.1.102 192.168.1.103"
IPMI_USER="admin"
IPMI_PASS="password"

for host in $IPMI_HOSTS; do
    echo "Temperature for $host:"
    ipmitool -I lanplus -H $host -U $IPMI_USER -P $IPMI_PASS sdr type temperature
done
```

### 6.3 Automated Health Check

Script for regular health checks:

```bash
#!/bin/bash
# health_check.sh
IPMI_HOSTS="192.168.1.101 192.168.1.102 192.168.1.103"
IPMI_USER="admin"
IPMI_PASS="password"

for host in $IPMI_HOSTS; do
    echo "Checking $host:"
    ipmitool -I lanplus -H $host -U $IPMI_USER -P $IPMI_PASS chassis status
    ipmitool -I lanplus -H $host -U $IPMI_USER -P $IPMI_PASS sel elist
done
```

## 7. Best Practices

1. Security: Use strong passwords and consider using IPMI over LAN+ for encrypted communications.
2. Scripting: Develop scripts for routine tasks and integrate with your monitoring systems.
3. Documentation: Keep detailed records of IPMI configurations for each server.
4. Regular Checks: Implement regular automated checks for system health.
5. Firmware Updates: Keep BMC firmware updated, but test updates thoroughly before applying to production systems.
6. Access Control: Implement strict access control policies for IPMI interfaces.
7. Monitoring Integration: Integrate IPMI data into your broader monitoring and alerting systems.

## 8. Troubleshooting

1. Connectivity Issues: 
   - Verify network settings
   - Check firewall rules (IPMI typically uses port 623)

2. Authentication Failures:
   - Verify username and password
   - Check privilege levels

3. Command Failures:
   - Ensure you have the correct privilege level
   - Some commands may not be supported by all hardware

4. Sensor Reading Errors:
   - May indicate hardware issues or sensor failures
   - Cross-reference with physical checks and other monitoring tools

Remember, while ipmitool is powerful, always use it cautiously, especially in production environments. Incorrect usage can lead to system downtime or data loss.