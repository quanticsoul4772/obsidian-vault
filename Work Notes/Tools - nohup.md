# Process Management with nohup

## Overview
nohup (no hang up) is a command to run processes that continue running after logging out or when the terminal is closed. It's essential for long-running processes and background tasks.

## Basic Usage

### Running Commands
```bash
# Basic usage
nohup command &

# Specify output file
nohup command > output.log 2>&1 &

# Run multiple commands
nohup bash -c 'command1; command2' &
```

### Process Management
```bash
# Check nohup processes
ps aux | grep nohup

# Find nohup output
ls -l nohup.out

# Kill nohup process
kill $(pgrep -f "nohup command")
```

## Automation Scripts

### Process Management
```bash
#!/bin/bash
# Manage long-running processes

start_process() {
    local command=$1
    local log_file=$2
    local pid_file=$3
    
    nohup $command > "$log_file" 2>&1 & 
    echo $! > "$pid_file"
}

stop_process() {
    local pid_file=$1
    if [ -f "$pid_file" ]; then
        kill $(cat "$pid_file")
        rm "$pid_file"
    fi
}

# Usage:
# start_process "python script.py" "script.log" "script.pid"
# stop_process "script.pid"
```

### Process Monitor
```bash
#!/bin/bash
# Monitor nohup processes

monitor_processes() {
    local processes_file="/etc/nohup_processes.conf"
    local log_dir="/var/log/nohup_monitor"
    
    mkdir -p "$log_dir"
    
    while read -r process; do
        if ! pgrep -f "$process" >/dev/null; then
            echo "Process not running: $process" >> "$log_dir/monitor.log"
            echo "Restarting process: $process" >> "$log_dir/monitor.log"
            nohup $process > "$log_dir/${process//\//_}.log" 2>&1 &
        fi
    done < "$processes_file"
}
```

## Best Practices

### Output Management
```bash
#!/bin/bash
# Rotate nohup logs

rotate_logs() {
    local log_dir=$1
    local max_size=$2  # in MB
    
    find "$log_dir" -name "nohup*.out" -size +"$max_size"M | while read log; do
        mv "$log" "${log}.$(date +%Y%m%d)"
        gzip "${log}.$(date +%Y%m%d)"
    done
}
```

### Process Wrapper
```bash
#!/bin/bash
# Wrapper for nohup processes

run_with_nohup() {
    local name=$1
    local command=$2
    local log_dir="/var/log/nohup"
    local pid_dir="/var/run/nohup"
    
    mkdir -p "$log_dir" "$pid_dir"
    
    # Start process
    nohup $command > "$log_dir/${name}.log" 2>&1 &
    echo $! > "$pid_dir/${name}.pid"
    
    # Set up log rotation
    cat > "/etc/logrotate.d/nohup-${name}" << EOF
$log_dir/${name}.log {
    rotate 7
    daily
    compress
    missingok
    notifempty
}
EOF
}
```

## Integration Examples

### Systemd Integration
```ini
[Unit]
Description=Nohup Process Manager
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/nohup_manager.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

### Process Status Dashboard
```bash
#!/bin/bash
# Generate process status report

generate_status() {
    echo "Nohup Process Status - $(date)"
    echo "=========================="
    
    ps aux | grep nohup | grep -v grep | while read line; do
        pid=$(echo $line | awk '{print $2}')
        cmd=$(echo $line | awk '{$1=$2=$3=$4=$5=$6=$7=$8=$9=$10=""; print $0}')
        runtime=$(ps -p $pid -o etime= 2>/dev/null)
        
        echo "PID: $pid"
        echo "Command: $cmd"
        echo "Runtime: $runtime"
        echo "--------------------------"
    done
}
```

## Troubleshooting

### Common Issues
1. Process Dies After Logout
```bash
# Check if process is properly detached
ps -ef | grep process_name
```

2. Output File Issues
```bash
# Check file permissions
ls -l nohup.out
# Fix permissions
chmod 644 nohup.out
```

3. Process Resource Usage
```bash
#!/bin/bash
# Monitor resource usage
while true; do
    ps -p $PID -o %cpu,%mem,cmd
    sleep 5
done
```

## Tags
#process-management #nohup #background-processes #system-admin