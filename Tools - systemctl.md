# systemctl

## Overview
systemctl is the primary command-line interface for managing and controlling systemd, the system and service manager for Linux. It enables system administrators to control the state of the systemd system and service manager, manage system services, and inspect system status.

## Basic Usage
```bash
# Check service status
systemctl status SERVICE_NAME

# Service control
systemctl start SERVICE_NAME
systemctl stop SERVICE_NAME
systemctl restart SERVICE_NAME
systemctl reload SERVICE_NAME

# Boot behavior
systemctl enable SERVICE_NAME
systemctl disable SERVICE_NAME

# System state commands
systemctl reboot
systemctl poweroff
systemctl suspend

# List units
systemctl list-units
systemctl list-units --type=service
systemctl list-units --state=failed

# View loaded unit files
systemctl list-unit-files

# Show dependencies
systemctl list-dependencies SERVICE_NAME

# Reload systemd manager
systemctl daemon-reload
```

## Automation Scripts
```bash
#!/bin/bash
# Service Monitor and Recovery Script

# Configuration
SERVICES_FILE="/etc/monitor/critical-services.txt"
LOG_FILE="/var/log/service-monitor.log"
ALERT_EMAIL="admin@example.com"

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Function to check and restart service if needed
monitor_service() {
    local service="$1"
    local status
    
    # Get service status
    status=$(systemctl is-active "$service")
    
    if [ "$status" != "active" ]; then
        log_message "WARNING: $service is $status"
        
        # Attempt restart
        log_message "Attempting to restart $service"
        systemctl restart "$service"
        
        # Check if restart successful
        if ! systemctl is-active "$service" >/dev/null 2>&1; then
            log_message "ERROR: Failed to restart $service"
            echo "Service $service failed on $(hostname)" | mail -s "Service Alert" "$ALERT_EMAIL"
            return 1
        else
            log_message "Successfully restarted $service"
        fi
    fi
    
    return 0
}

# Generate status report in JSON format
generate_report() {
    local report_file="/var/log/service-status.json"
    echo "{" > "$report_file"
    echo "  \"timestamp\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\"," >> "$report_file"
    echo "  \"hostname\": \"$(hostname)\"," >> "$report_file"
    echo "  \"services\": [" >> "$report_file"
    
    local first=true
    while IFS= read -r service; do
        [ -z "$service" ] && continue
        
        if ! $first; then
            echo "," >> "$report_file"
        fi
        
        local status=$(systemctl is-active "$service")
        local enabled=$(systemctl is-enabled "$service")
        
        echo "    {" >> "$report_file"
        echo "      \"name\": \"$service\"," >> "$report_file"
        echo "      \"status\": \"$status\"," >> "$report_file"
        echo "      \"enabled\": \"$enabled\"" >> "$report_file"
        echo -n "    }" >> "$report_file"
        
        first=false
    done < "$SERVICES_FILE"
    
    echo "" >> "$report_file"
    echo "  ]" >> "$report_file"
    echo "}" >> "$report_file"
}

# Main execution
log_message "Starting service monitoring"

while IFS= read -r service; do
    [ -z "$service" ] && continue
    monitor_service "$service"
done < "$SERVICES_FILE"

generate_report
```

## Integration Examples
### Prometheus Integration
```yaml
# prometheus.yml configuration for systemd monitoring
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']

# Alert rules
groups:
  - name: systemd
    rules:
      - alert: ServiceDown
        expr: node_systemd_unit_state{state="active"} == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.name }} is down"
```

### Monitoring Script Output Format
```json
{
  "timestamp": "2024-12-16T10:30:00Z",
  "hostname": "server-01",
  "services": [
    {
      "name": "nginx",
      "status": "active",
      "enabled": "enabled"
    }
  ]
}
```

## Troubleshooting Guide
Common issues and their solutions:

1. Service Fails to Start
```bash
# View detailed service status
systemctl status SERVICE_NAME -l

# Check service logs
journalctl -u SERVICE_NAME -n 100 --no-pager

# Verify service configuration
systemctl cat SERVICE_NAME

# Check service dependencies
systemctl list-dependencies SERVICE_NAME --all
```

2. Service Starts but Crashes
```bash
# Monitor real-time logs
journalctl -u SERVICE_NAME -f

# Check resource limits
systemctl show SERVICE_NAME | grep -E 'Limit|Memory|CPU'

# View process tree
systemctl status SERVICE_NAME --full
```

3. Boot Problems
```bash
# Analyze boot time
systemd-analyze blame

# Check critical chain
systemd-analyze critical-chain

# List failed units
systemctl --failed
```

## Best Practices
1. Service Management
- Always run daemon-reload after modifying unit files
- Use override files instead of editing unit files directly
- Document service dependencies
- Implement proper monitoring
- Set appropriate resource limits

2. Security
- Review service permissions regularly
- Use service hardening options
- Follow principle of least privilege
- Implement resource limits

3. Monitoring
- Set up automated service monitoring
- Configure alerting for critical services
- Maintain service logs
- Regular status checks
- Monitor resource usage

4. Documentation
- Keep service configuration documented
- Maintain dependency documentation
- Log configuration changes
- Document troubleshooting procedures

## Tags
#systemd #service-management #linux #sysadmin #monitoring #automation