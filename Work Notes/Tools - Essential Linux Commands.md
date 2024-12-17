# Essential Linux Commands Guide

## Overview
This guide provides a comprehensive reference for essential Linux commands commonly used in system administration and HPC environments. Commands are organized by category and include practical examples, options, and best practices.

## System Information

### System and Hardware
```bash
# System information
uname -a          # All system information
hostnamectl       # System and OS information
lscpu             # CPU information
free -h           # Memory usage in human-readable format
df -h             # Disk usage in human-readable format
lsblk             # Block device information
dmidecode         # Hardware information

# Process information
top               # Real-time process monitoring
htop              # Interactive process viewer
ps aux            # List all processes
pstree            # Display process tree
```

### Network Information
```bash
# Network interfaces
ip addr           # IP address information
ip link           # Network interface information
iwconfig          # Wireless interface information
nmcli            # NetworkManager control

# Network connectivity
ping             # Test network connectivity
traceroute       # Trace packet route
netstat -tuln    # List listening ports
ss -tuln         # Socket statistics
```

## File Operations

### Basic File Management
```bash
# File operations
ls -la           # List files with details
cp -r src dst    # Copy files/directories recursively
mv src dst       # Move/rename files
rm -rf dir       # Remove files/directories recursively
mkdir -p dir     # Create directory with parents
chmod 755 file   # Change file permissions
chown user file  # Change file ownership

# File searching
find . -name "pattern"     # Find files by name
locate filename           # Find files in database
which command            # Show command path
whereis command          # Show binary/man locations
```

### File Content
```bash
# Content viewing
cat file         # Display file content
less file        # Page through file
head -n 10 file # Show first 10 lines
tail -f file    # Follow file updates
grep pattern file # Search file content
zcat file.gz    # View compressed file

# File manipulation
sort file        # Sort file content
uniq            # Remove duplicates
cut -d: -f1     # Cut specific fields
sed 's/old/new/' # Stream editor
awk '{print $1}' # Pattern processing
```

## Process Management

### Process Control
```bash
# Process management
ps aux | grep process  # Find process
kill -9 PID           # Force kill process
pkill process_name    # Kill by process name
nice -n 19 command    # Run with low priority
renice -n 19 -p PID   # Change process priority
nohup command &       # Run immune to hangups
```

### Job Control
```bash
# Job management
jobs              # List background jobs
bg %1             # Send job to background
fg %1             # Bring job to foreground
ctrl+z            # Suspend current job
ctrl+c            # Terminate current job
```

## System Administration

### User Management
```bash
# User operations
useradd username  # Create user
usermod -aG group user # Add user to group
passwd username   # Set user password
su - username    # Switch user
sudo command     # Execute as root
who              # Show logged in users
w                # Show who is doing what
```

### Service Management
```bash
# Systemd services
systemctl status service  # Check service status
systemctl start service  # Start service
systemctl enable service # Enable at boot
journalctl -u service   # View service logs
systemctl list-units    # List all units
```

## Storage Management

### Disk Operations
```bash
# Disk management
fdisk -l         # List disk partitions
parted -l        # List partitions (GPT)
mkfs.ext4 /dev/sdX # Create filesystem
mount /dev/sdX /mnt # Mount filesystem
umount /mnt      # Unmount filesystem
swapon --show    # Show swap usage
```

### Logical Volume Management
```bash
# LVM operations
pvdisplay        # Show physical volumes
vgdisplay        # Show volume groups
lvdisplay        # Show logical volumes
lvextend        # Extend logical volume
resize2fs        # Resize filesystem
```

## Performance Monitoring

### Resource Monitoring
```bash
# System resources
vmstat 1         # Virtual memory stats
iostat 1         # IO statistics
sar -u 1 3       # CPU utilization
mpstat -P ALL 1  # CPU usage by core
iotop            # IO monitoring
```

### Network Monitoring
```bash
# Network monitoring
iftop            # Network usage by interface
nethogs          # Network usage by process
tcpdump          # Packet capture
nload            # Network load monitor
```

## System Maintenance

### Package Management
```bash
# APT (Debian/Ubuntu)
apt update       # Update package list
apt upgrade      # Upgrade packages
apt install pkg  # Install package
apt remove pkg   # Remove package

# DNF (RHEL/Rocky)
dnf update       # Update package list
dnf upgrade      # Upgrade packages
dnf install pkg  # Install package
dnf remove pkg   # Remove package
```

### Log Management
```bash
# Log viewing
journalctl      # View systemd logs
tail -f /var/log/syslog # Follow syslog
zgrep pattern /var/log/syslog.*.gz # Search compressed logs
logger "message" # Add log entry
```

## Best Practices

### Command Usage
1. Always use full paths in scripts
2. Quote variables to handle spaces
3. Use sudo only when necessary
4. Check command exit status
5. Use manpages for detailed help

### Safety Measures
1. Use rm -i for interactive removal
2. Preview sed/awk changes
3. Backup before major changes
4. Test commands on sample data
5. Use screen/tmux for long operations

## Automation Tips

### Script Development
```bash
# Common script header
#!/bin/bash
set -euo pipefail
IFS=$'\n\t'

# Error handling
trap 'echo "Error on line $LINENO"' ERR

# Logging
exec 1> >(logger -s -t $(basename $0)) 2>&1
```

### Command Combinations
```bash
# Useful combinations
find . -type f -exec chmod 644 {} \;  # Change all file permissions
tar czf - dir | ssh host "cd /dst && tar xzf -"  # Remote copy
dd if=/dev/zero of=file bs=1M count=100  # Create test file
```

## Troubleshooting Guide

### Common Issues
1. Permission denied
   - Check file permissions
   - Verify user/group ownership
   - Use sudo if necessary

2. Command not found
   - Check PATH variable
   - Install required package
   - Use full command path

3. Resource exhaustion
   - Check disk space
   - Monitor memory usage
   - Review process limits

## Related Documentation
- [[Tools - Performance Monitoring Scripts]]
- [[Troubleshooting Decision Tree]]
- [[SysAdmin Hub]]

## Tags
#linux #commands #system-admin #reference #troubleshooting