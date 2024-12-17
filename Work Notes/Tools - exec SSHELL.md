# exec SSHELL

## Overview
The `exec sshell` command is a remote shell execution tool typically used in IPMI (Intelligent Platform Management Interface) environments. It provides secure shell access to remote systems through the IPMI interface, allowing administrators to access systems even when the operating system is not responsive.

## Basic Usage
```bash
# Basic SSHELL connection
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD sol activate

# Execute specific command
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD shell command "COMMAND"

# Configure SSHELL settings
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD shell set timeout 120

# Check SSHELL status
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD shell info

# Deactivate SSHELL session
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD sol deactivate
```

## Automation Scripts
```bash
#!/bin/bash
# Remote System Management Script

# Configuration
CONFIG_FILE="/etc/ipmi/hosts.conf"
LOG_FILE="/var/log/remote-shell.log"
ALERT_EMAIL="admin@example.com"
TIMEOUT=120

# Function to log messages
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Function to execute remote command
execute_remote_command() {
    local host="$1"
    local username="$2"
    local password="$3"
    local command="$4"
    local output_file="/tmp/sshell_${host//[.:]/}_$(date +%s).json"
    
    log_message "Executing command on $host: $command"
    
    # Execute command and capture output
    if ! ipmitool -I lanplus -H "$host" -U "$username" -P "$password" shell command "$command" > "$output_file" 2>&1; then
        log_message "ERROR: Command execution failed on $host"
        echo "Remote command failed on $host" | mail -s "Remote Execution Alert" "$ALERT_EMAIL"
        return 1
    fi
    
    # Generate JSON output
    {
        echo "{"
        echo "  \"timestamp\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\","
        echo "  \"host\": \"$host\","
        echo "  \"command\": \"$command\","
        echo "  \"status\": \"success\","
        echo "  \"output\": \"$(cat "$output_file" | sed 's/"/\\"/g')\""
        echo "}"
    } > "${output_file}.json"
    
    rm "$output_file"
    return 0
}

# Function to check host accessibility
check_host() {
    local host="$1"
    local username="$2"
    local password="$3"
    
    log_message "Checking host accessibility: $host"
    
    if ! ipmitool -I lanplus -H "$host" -U "$username" -P "$password" shell info >/dev/null 2>&1; then
        log_message "WARNING: Unable to reach $host"
        return 1
    fi
    
    return 0
}

# Function to generate status report
generate_report() {
    local report_file="/var/log/sshell-status.json"
    echo "{" > "$report_file"
    echo "  \"timestamp\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\"," >> "$report_file"
    echo "  \"hosts\": [" >> "$report_file"
    
    local first=true
    while IFS=',' read -r host username password; do
        [ -z "$host" ] && continue
        
        if ! $first; then
            echo "," >> "$report_file"
        fi
        first=false
        
        if check_host "$host" "$username" "$password"; then
            status="available"
        else
            status="unreachable"
        fi
        
        echo "    {" >> "$report_file"
        echo "      \"host\": \"$host\"," >> "$report_file"
        echo "      \"status\": \"$status\"," >> "$report_file"
        echo "      \"last_check\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\"" >> "$report_file"
        echo -n "    }" >> "$report_file"
    done < "$CONFIG_FILE"
    
    echo -e "\n  ]\n}" >> "$report_file"
}

# Main execution
log_message "Starting remote system management"
generate_report
```

## Integration Examples
### Prometheus Integration
```yaml
# prometheus.yml configuration for IPMI monitoring
scrape_configs:
  - job_name: 'ipmi'
    static_configs:
      - targets: ['localhost:9290']
    metrics_path: '/metrics'
    scheme: 'http'

# Alert rules for IPMI connectivity
groups:
  - name: ipmi_alerts
    rules:
      - alert: IPMIHostUnreachable
        expr: ipmi_host_status == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "IPMI host unreachable"
          description: "Unable to connect to {{ $labels.host }} via IPMI"
```

## Troubleshooting Guide
1. Connection Issues
```bash
# Check IPMI connectivity
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD chassis status

# Verify network access
ping HOST

# Test IPMI port accessibility
nc -zv HOST 623

# Check IPMI logs
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD sel list
```

2. Authentication Problems
```bash
# Verify credentials
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD user list

# Check user privileges
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD channel getaccess 1

# Reset authentication
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD session zero
```

3. Session Management
```bash
# List active sessions
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD session info all

# Clear stuck sessions
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD session clear all

# Check session timeout
ipmitool -I lanplus -H HOST -U USERNAME -P PASSWORD shell get timeout
```

## Best Practices
1. Security
- Use strong passwords
- Change default credentials
- Implement access control
- Monitor failed attempts
- Secure IPMI network

2. Session Management
- Set appropriate timeouts
- Clear inactive sessions
- Monitor active sessions
- Document session policies
- Implement session limits

3. Monitoring
- Regular connectivity checks
- Monitor response times
- Track failed attempts
- Log command execution
- Monitor system status

4. Documentation
- Keep credential inventory
- Document host configurations
- Log access attempts
- Record configuration changes
- Maintain troubleshooting guides

## Tags
#ipmi #remote-management #system-administration #security #monitoring