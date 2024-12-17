# dmesg

## Overview
dmesg (diagnostic message) is a command that displays the kernel ring buffer content on Unix-like operating systems. It's a powerful tool for system administrators and users to view and analyze system logs, especially those related to hardware, drivers, and kernel operations.

Key features:
- Displays kernel ring buffer messages
- Shows boot-time messages
- Logs hardware detection and driver initialization
- Records system errors and warnings

## Basic Usage
```bash
# Display all kernel messages
dmesg

# Display human-readable timestamps
dmesg -T
dmesg -H    # Adds colors and other human-friendly formatting

# Follow new messages in real time
dmesg -w

# Clear the kernel ring buffer (after displaying)
sudo dmesg -c

# Show only error and warning messages
dmesg -l err,warn

# Show only kernel facility messages
dmesg -f kern

# Search for specific text
dmesg | grep USB

# Show messages since last boot
dmesg -b

# Show kernel log level and timestamp
dmesg -x

# Display available log levels and facilities
dmesg -L
```

## Advanced Usage
```bash
# Control kernel ring buffer size
sudo sysctl -w kernel.printk_buffer_len=128000

# Redirect kernel messages to file
sudo dmesg > kernel_messages.txt

# View messages from specific time range
dmesg --since "1 hour ago"
dmesg --until "2 minutes ago"
```

## Automation Scripts
```bash
#!/bin/bash
# Kernel Event Monitor and Logger

# Configuration
LOG_DIR="/var/log/kernel-events"
ALERT_EMAIL="admin@example.com"
MAX_LOG_SIZE_MB=100

# Create log directory if it doesn't exist
mkdir -p "$LOG_DIR"

# Function to rotate logs if they exceed size limit
rotate_logs() {
    local log_size=$(du -sm "$LOG_DIR" | cut -f1)
    if [ "$log_size" -gt "$MAX_LOG_SIZE_MB" ]; then
        local archive_name="kernel_logs_$(date +%Y%m%d_%H%M%S).tar.gz"
        tar -czf "$LOG_DIR/archives/$archive_name" "$LOG_DIR"/*.log
        rm "$LOG_DIR"/*.log
    fi
}

# Function to parse and format dmesg output
parse_dmesg() {
    local output_file="$LOG_DIR/kernel_events_$(date +%Y%m%d).log"
    local temp_file=$(mktemp)

    dmesg -T --level=err,crit,alert,emerg | while read -r line; do
        echo "[$(date '+%Y-%m-%d %H:%M:%S')] $line" >> "$temp_file"
        
        # Check for critical patterns
        case "$line" in
            *"Out of memory"*|*"Kernel panic"*|*"Hardware error"*)
                echo "[CRITICAL] $line" | mail -s "Kernel Alert on $(hostname)" "$ALERT_EMAIL"
                ;;
        esac
    done

    # Append new entries to log file
    cat "$temp_file" >> "$output_file"
    rm "$temp_file"

    # Generate JSON summary
    {
        echo "{"
        echo "  \"timestamp\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\","
        echo "  \"hostname\": \"$(hostname)\","
        echo "  \"events\": {"
        echo "    \"errors\": $(dmesg | grep -c "error"),"
        echo "    \"warnings\": $(dmesg | grep -c "warning"),"
        echo "    \"critical\": $(dmesg | grep -ci "critical")"
        echo "  }"
        echo "}"
    } > "$LOG_DIR/summary.json"
}

# Main execution
rotate_logs
parse_dmesg
```

## Integration Examples
### Prometheus Integration
```yaml
# prometheus.yml snippet for kernel metrics
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
    params:
      collect[]:
        - kernel
        - dmesg

# Alert rules for kernel events
groups:
  - name: kernel_alerts
    rules:
      - alert: KernelError
        expr: node_dmesg_errors_total > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Kernel errors detected"
```

## Interpreting Output
1. Boot Process
   - Initial messages show boot process
   - Hardware detection sequence
   - Driver initialization events
   - Resource allocation

2. Hardware Issues
   - Look for terms like "error", "fail"
   - Hardware component names
   - IRQ conflicts
   - Memory allocation issues

3. Driver Problems
   - Failed driver loading
   - Driver errors or warnings
   - Module initialization failures

4. Performance Issues
   - CPU throttling messages
   - Memory allocation failures
   - Disk I/O errors
   - System resource limits

## Troubleshooting Guide
1. System Boot Issues
```bash
# Check boot messages
dmesg | grep -i "error"
dmesg | grep -i "fail"

# Analyze hardware detection
dmesg | grep -i "detected"

# Check driver loading
dmesg | grep -i "driver"
```

2. Hardware Problems
```bash
# Memory issues
dmesg | grep -i "memory"

# Disk problems
dmesg | grep -i "sd[a-z]"

# USB issues
dmesg | grep -i "usb"

# Network interface problems
dmesg | grep -i "eth\|wlan"
```

## Best Practices
1. Monitoring
   - Regularly check dmesg output for errors
   - Implement automated monitoring
   - Set up alerts for critical messages
   - Keep logs rotated and archived
   - Monitor hardware-related messages

2. Analysis
   - Use timestamps for accurate tracking
   - Filter messages by priority
   - Correlate with system issues
   - Document patterns and solutions
   - Maintain historical logs

3. Integration
   - Configure with monitoring systems
   - Set up automated reporting
   - Configure alert thresholds
   - Use structured logging
   - Implement log analysis

## Important Notes
1. Root privileges may be required to view all messages
2. Kernel ring buffer has limited size; older messages get overwritten
3. For persistent logs, check /var/log/kern.log or use journalctl
4. Essential for hardware and driver troubleshooting
5. Consider using with other monitoring tools for comprehensive system oversight

## Tags
#kernel #system-logs #troubleshooting #monitoring #hardware #diagnostics