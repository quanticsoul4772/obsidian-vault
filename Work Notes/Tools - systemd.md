# systemd

## Overview
systemd is the init system and system manager that has become the de facto standard for Linux systems. It handles the boot process after the initial Linux kernel is loaded, manages system resources and services, and handles system shutdown. systemd is designed to provide aggressive parallelization capabilities, uses socket and D-Bus activation for starting services, offers on-demand starting of daemons, and tracks processes using Linux control groups.

## Basic Usage
```bash
# View system boot time information
systemd-analyze

# Show boot time of all units
systemd-analyze blame

# View unit dependencies
systemd-analyze critical-chain

# Check system status
systemctl status

# List failed units
systemctl --failed

# Reload systemd manager
systemctl daemon-reload

# View system logs
journalctl

# View kernel messages
journalctl -k

# View logs for specific unit
journalctl -u unit_name

# View logs since last boot
journalctl -b
```

## Automation Scripts
```bash
#!/bin/bash
# Systemd Health Check and Monitoring Script

# Configuration
LOG_DIR="/var/log/systemd-monitor"
ALERT_EMAIL="admin@example.com"
CRITICAL_UNITS=("/etc/systemd/critical-units.txt")
JOURNAL_RETENTION="2weeks"

# Create log directory
mkdir -p "$LOG_DIR"

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_DIR/monitor.log"
}

# Function to check systemd health
check_systemd_health() {
    local report_file="$LOG_DIR/health_$(date +%Y%m%d_%H%M%S).json"
    
    # Get failed units
    failed_units=$(systemctl --failed --no-legend | awk '{print $1}')
    
    # Get system state
    system_state=$(systemctl is-system-running)
    
    # Generate JSON report
    {
        echo "{"
        echo "  \"timestamp\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\","
        echo "  \"hostname\": \"$(hostname)\","
        echo "  \"system_state\": \"$system_state\","
        echo "  \"failed_units\": ["
        if [ -n "$failed_units" ]; then
            echo "$failed_units" | while read -r unit; do
                echo "    \"$unit\","
            done | sed '$ s/,$//'
        fi
        echo "  ],"
        echo "  \"boot_time\": \"$(systemd-analyze | grep 'Startup finished in' | sed 's/Startup finished in //')\","
        echo "  \"journal_size\": \"$(journalctl --disk-usage | awk '{print $7}')\""
        echo "}"
    } > "$report_file"
    
    # Alert on failed units
    if [ -n "$failed_units" ]; then
        log_message "WARNING: Found failed units"
        echo "Failed units on $(hostname):" | mail -s "Systemd Alert" "$ALERT_EMAIL"
        echo "$failed_units" | mail -s "Failed Units Report" "$ALERT_EMAIL"
    fi
}

# Function to analyze boot performance
analyze_boot() {
    local analysis_file="$LOG_DIR/boot_analysis_$(date +%Y%m%d).txt"
    
    {
        echo "Boot Time Analysis - $(date '+%Y-%m-%d %H:%M:%S')"
        echo "----------------------------------------"
        echo "Overall Boot Time:"
        systemd-analyze
        echo
        echo "Top 10 Time-Consuming Units:"
        systemd-analyze blame | head -n 10
        echo
        echo "Critical Chain:"
        systemd-analyze critical-chain
    } > "$analysis_file"
}

# Function to manage journal
manage_journal() {
    # Rotate journals
    journalctl --rotate
    
    # Vacuum old entries
    journalctl --vacuum-time="$JOURNAL_RETENTION"
    
    # Check journal health
    journalctl --verify
}

# Function to monitor critical units
monitor_critical_units() {
    while IFS= read -r unit; do
        [ -z "$unit" ] && continue
        
        status=$(systemctl is-active "$unit")
        if [ "$status" != "active" ]; then
            log_message "CRITICAL: Unit $unit is $status"
            echo "Critical unit $unit is $status on $(hostname)" | mail -s "Critical Unit Alert" "$ALERT_EMAIL"
        fi
    done < "$CRITICAL_UNITS"
}

# Main execution
log_message "Starting systemd health check"
check_systemd_health
analyze_boot
manage_journal
monitor_critical_units
```

## Integration Examples
### Prometheus Integration
```yaml
# prometheus.yml configuration for systemd monitoring
scrape_configs:
  - job_name: 'systemd'
    static_configs:
      - targets: ['localhost:9100']
    metrics_path: '/metrics'
    params:
      collect[]:
        - systemd

# Alert rules for systemd monitoring
groups:
  - name: systemd_alerts
    rules:
      - alert: SystemdUnitFailed
        expr: node_systemd_unit_state{state="failed"} > 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Systemd unit failed"
          description: "Unit {{ $labels.name }} has failed"

      - alert: SystemdInDegradedState
        expr: node_systemd_system_running != 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "System in degraded state"
          description: "Systemd is not in running state"
```

## Troubleshooting Guide
1. Unit Problems
```bash
# Check unit status
systemctl status unit_name

# View unit configuration
systemctl cat unit_name

# View unit dependencies
systemctl list-dependencies unit_name

# Check unit file syntax
systemd-analyze verify unit_name
```

2. Boot Issues
```bash
# Analyze slow boot
systemd-analyze blame

# Check critical chain
systemd-analyze critical-chain

# Debug boot process
systemd-analyze plot > boot.svg

# Check for failed units during boot
systemctl --failed
```

3. Journal Problems
```bash
# Check journal errors
journalctl -p err..alert

# View boots
journalctl --list-boots

# Check disk usage
journalctl --disk-usage

# Verify journal integrity
journalctl --verify
```

## Best Practices
1. System Management
- Regular health checks
- Monitor boot performance
- Track failed units
- Manage journal size
- Monitor critical services

2. Performance
- Optimize boot time
- Track resource usage
- Monitor dependencies
- Manage service load
- Set appropriate timeouts

3. Maintenance
- Regular journal rotation
- Clean old logs
- Update unit files properly
- Monitor system state
- Track configuration changes

4. Security
- Restrict unit file permissions
- Monitor privileged services
- Track authentication failures
- Audit service access
- Review unit dependencies

5. Documentation
- Document unit configurations
- Track system changes
- Maintain troubleshooting guides
- Record common issues
- Document recovery procedures

## Tags
#systemd #init-system #system-management #service-management #monitoring