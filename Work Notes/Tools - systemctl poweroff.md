# systemctl poweroff

## Overview
`systemctl poweroff` is a systemd command used to safely shut down and power off the system. It initiates a clean shutdown process, stopping all services in the correct order, syncing filesystems, and finally powering off the machine. This command is part of systemd's power management functionality and is preferred over traditional commands like `shutdown` or `poweroff` on systemd-based systems.

## Basic Usage
```bash
# Basic power off command
systemctl poweroff

# Schedule power off in 5 minutes
systemctl poweroff --delay=5min

# Power off with wall message
systemctl poweroff --message="System maintenance shutdown"

# Cancel scheduled power off
systemctl cancel

# Force immediate power off (not recommended)
systemctl poweroff -i

# Check if system is scheduled for power off
systemctl status shutdown.target
```

## Automation Scripts
```bash
#!/bin/bash
# Safe System Shutdown Script

# Configuration
LOG_FILE="/var/log/system-shutdown.log"
ALERT_EMAIL="admin@example.com"
CRITICAL_SERVICES=("/etc/shutdown/critical-services.txt")
PRE_SHUTDOWN_SCRIPTS="/etc/shutdown/pre-shutdown.d"
GRACE_PERIOD=300  # 5 minutes

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Function to check active sessions
check_active_sessions() {
    local active_users=$(who | wc -l)
    if [ "$active_users" -gt 0 ]; then
        log_message "WARNING: Found $active_users active user sessions"
        who | mail -s "Active Sessions During Shutdown - $(hostname)" "$ALERT_EMAIL"
        return 1
    fi
    return 0
}

# Function to check critical services
check_critical_services() {
    local errors=0
    while IFS= read -r service; do
        [ -z "$service" ] && continue
        
        if systemctl is-active --quiet "$service"; then
            log_message "Preparing to stop $service"
            
            # Attempt graceful stop
            if ! systemctl stop "$service"; then
                log_message "ERROR: Failed to stop $service"
                ((errors++))
            fi
        fi
    done < "$CRITICAL_SERVICES"
    
    return "$errors"
}

# Function to flush filesystem buffers
flush_filesystems() {
    log_message "Syncing filesystems"
    sync
    
    # Check for busy mounts
    local busy_mounts=$(lsof / 2>/dev/null | grep -v "COMMAND" | wc -l)
    if [ "$busy_mounts" -gt 0 ]; then
        log_message "WARNING: Found busy mounts"
        return 1
    fi
    return 0
}

# Function to generate shutdown report
generate_shutdown_report() {
    local report_file="/tmp/shutdown_report_$(date +%Y%m%d_%H%M%S).json"
    
    {
        echo "{"
        echo "  \"timestamp\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\","
        echo "  \"hostname\": \"$(hostname)\","
        echo "  \"uptime\": \"$(uptime -p)\","
        echo "  \"active_services\": ["
        systemctl list-units --type=service --state=running --no-legend | \
            awk '{printf "    \"%s\"%s\n", $1, NR==1?",":""}' | \
            sed '$ s/,$//'
        echo "  ],"
        echo "  \"filesystem_status\": {"
        df -h --output=source,target,pcent | tail -n +2 | \
            awk '{printf "    \"%s\": \"%s\"%s\n", $2, $3, NR==1?",":""}' | \
            sed '$ s/,$//'
        echo "  }"
        echo "}"
    } > "$report_file"
    
    # Send report via email
    cat "$report_file" | mail -s "Shutdown Report - $(hostname)" "$ALERT_EMAIL"
}

# Function to execute pre-shutdown scripts
run_pre_shutdown_scripts() {
    if [ -d "$PRE_SHUTDOWN_SCRIPTS" ]; then
        for script in "$PRE_SHUTDOWN_SCRIPTS"/*; do
            if [ -x "$script" ]; then
                log_message "Executing pre-shutdown script: $script"
                if ! "$script"; then
                    log_message "ERROR: Pre-shutdown script failed: $script"
                    return 1
                fi
            fi
        done
    fi
    return 0
}

# Main execution
log_message "Initiating shutdown procedure"

# Check for force flag
FORCE=0
if [ "$1" == "--force" ]; then
    FORCE=1
    log_message "Force shutdown requested"
fi

# Execute shutdown steps
run_pre_shutdown_scripts
if [ $? -ne 0 ] && [ $FORCE -eq 0 ]; then
    log_message "ERROR: Pre-shutdown scripts failed, aborting"
    exit 1
fi

if ! check_active_sessions && [ $FORCE -eq 0 ]; then
    log_message "ERROR: Active sessions detected, aborting"
    exit 1
fi

check_critical_services
flush_filesystems
generate_shutdown_report

log_message "Initiating system poweroff"
systemctl poweroff --message="System shutdown completed successfully"
```

## Integration Examples
### Prometheus Alert Rules
```yaml
# Alert rules for system shutdown monitoring
groups:
  - name: shutdown_alerts
    rules:
      - alert: UnplannedShutdown
        expr: node_system_power_state{state="shutdown"} == 1 
              unless on(instance) shutdown_scheduled == 1
        labels:
          severity: critical
        annotations:
          summary: "Unplanned shutdown detected"
          description: "System shutdown initiated without scheduling"

      - alert: ShutdownWithActiveSessions
        expr: node_system_power_state{state="shutdown"} == 1 
              and node_active_sessions > 0
        labels:
          severity: warning
        annotations:
          summary: "Shutdown with active sessions"
          description: "System shutdown initiated with active user sessions"
```

## Troubleshooting Guide
1. Failed Shutdown
```bash
# Check system logs
journalctl -b -1 -n 1000

# View last shutdown reason
last -x shutdown

# Check for stuck processes
ps aux --forest

# View system state
systemctl status
```

2. Service Hang During Shutdown
```bash
# List running services
systemctl list-units --type=service --state=running

# Check specific service status
systemctl status service_name

# Force stop service
systemctl kill service_name
```

3. Filesystem Issues
```bash
# Check mounted filesystems
mount | grep -v '^/dev'

# Check for busy files
lsof / | grep -v COMMAND

# Force sync filesystems
sync; sync; sync
```

## Best Practices
1. Pre-Shutdown Procedures
- Verify no critical processes running
- Check for active user sessions
- Save system state
- Stop services gracefully
- Document shutdown reason

2. System Health
- Monitor filesystem status
- Check service states
- Verify system resources
- Document system state
- Generate status reports

3. Security
- Authorize shutdown requests
- Log shutdown events
- Monitor unplanned shutdowns
- Track failed attempts
- Verify user permissions

4. Recovery
- Create shutdown logs
- Document shutdown steps
- Maintain recovery procedures
- Test emergency procedures
- Keep contact information updated

5. Documentation
- Record shutdown reasons
- Track shutdown patterns
- Document special procedures
- Maintain service dependencies
- Keep troubleshooting guides

## Tags
#systemd #shutdown #power-management #system-administration #maintenance