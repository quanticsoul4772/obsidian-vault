# iperf

## Overview
iperf is a network performance measurement tool that can test maximum TCP and UDP bandwidth performance. It allows for the tuning of various parameters and characteristics of the network protocol being used for testing. iperf3, the latest version, is a complete rewrite that includes additional functionality and a simplified API.

## Basic Usage
```bash
# Start iperf server
iperf3 -s

# Run client test to a server
iperf3 -c SERVER_IP

# Specific port test
iperf3 -c SERVER_IP -p 5201

# UDP test
iperf3 -c SERVER_IP -u

# Test with specific bandwidth
iperf3 -c SERVER_IP -b 100M

# Bidirectional test
iperf3 -c SERVER_IP --bidir

# Multiple parallel streams
iperf3 -c SERVER_IP -P 4

# Test for specific duration
iperf3 -c SERVER_IP -t 30

# JSON output format
iperf3 -c SERVER_IP --json

# Format as Mbits/sec
iperf3 -c SERVER_IP --format m
```

## Automation Scripts
```bash
#!/bin/bash
# Network Performance Testing and Monitoring Script

# Configuration
TARGETS_FILE="/etc/network/test-targets.txt"
LOG_DIR="/var/log/network-tests"
ALERT_EMAIL="admin@example.com"
BANDWIDTH_THRESHOLD=100  # Mbps
ALERT_ON_FAILURE=true

# Create log directory if it doesn't exist
mkdir -p "$LOG_DIR"

# Function to log messages
log_message() {
    local level=$1
    local message=$2
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message" | tee -a "$LOG_DIR/network-tests.log"
}

# Function to run network test
run_network_test() {
    local target=$1
    local output_file="$LOG_DIR/test_${target//[.:]/}_$(date +%Y%m%d_%H%M%S).json"
    
    # Run iperf3 test with JSON output
    if ! iperf3 -c "$target" -J -t 10 --format m > "$output_file" 2>&1; then
        log_message "ERROR" "Test to $target failed"
        if [ "$ALERT_ON_FAILURE" = true ]; then
            echo "Network test to $target failed on $(hostname)" | mail -s "Network Test Alert" "$ALERT_EMAIL"
        fi
        return 1
    fi
    
    # Parse results and check threshold
    local bandwidth=$(jq '.end.sum_received.bits_per_second' "$output_file" 2>/dev/null)
    if [ -n "$bandwidth" ]; then
        bandwidth=$(echo "$bandwidth / 1000000" | bc)  # Convert to Mbps
        if (( $(echo "$bandwidth < $BANDWIDTH_THRESHOLD" | bc -l) )); then
            log_message "WARNING" "Bandwidth to $target below threshold: ${bandwidth}Mbps"
            if [ "$ALERT_ON_FAILURE" = true ]; then
                echo "Low bandwidth detected to $target: ${bandwidth}Mbps" | mail -s "Bandwidth Alert" "$ALERT_EMAIL"
            fi
        fi
    fi
    
    return 0
}

# Function to generate summary report
generate_report() {
    local report_file="$LOG_DIR/summary_$(date +%Y%m%d).json"
    
    echo "{" > "$report_file"
    echo "  \"timestamp\": \"$(date -u '+%Y-%m-%dT%H:%M:%SZ')\"," >> "$report_file"
    echo "  \"tests\": [" >> "$report_file"
    
    local first=true
    while IFS= read -r target; do
        [ -z "$target" ] && continue
        
        local latest_test=$(ls -t "$LOG_DIR"/test_${target//[.:]/}_*.json 2>/dev/null | head -n1)
        if [ -n "$latest_test" ]; then
            if ! $first; then
                echo "," >> "$report_file"
            fi
            first=false
            
            jq -c "{
                target: \"$target\",
                timestamp: .start.timestamp.time,
                bandwidth_mbps: (.end.sum_received.bits_per_second / 1000000),
                retransmits: .end.sum_sent.retransmits,
                jitter_ms: .end.sum.jitter_ms,
                lost_packets: .end.sum.lost_packets
            }" "$latest_test" >> "$report_file"
        fi
    done < "$TARGETS_FILE"
    
    echo -e "\n  ]\n}" >> "$report_file"
}

# Main execution
log_message "INFO" "Starting network performance tests"

while IFS= read -r target; do
    [ -z "$target" ] && continue
    log_message "INFO" "Testing connection to $target"
    run_network_test "$target"
done < "$TARGETS_FILE"

generate_report
```

## Integration Examples
### Prometheus Integration
```yaml
# prometheus.yml configuration for network metrics
scrape_configs:
  - job_name: 'network_performance'
    static_configs:
      - targets: ['localhost:9100']
    metrics_path: '/metrics'
    scheme: 'http'

# Alert rules for network performance
groups:
  - name: network_performance
    rules:
      - alert: LowNetworkBandwidth
        expr: network_bandwidth_mbps < 100
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low network bandwidth detected"
          description: "Bandwidth below 100Mbps for {{ $labels.target }}"
      
      - alert: HighPacketLoss
        expr: network_packet_loss_percent > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High packet loss detected"
          description: "Packet loss > 1% for {{ $labels.target }}"
```

## Troubleshooting Guide
1. Connection Issues
```bash
# Test basic connectivity
ping TARGET_IP

# Check port availability
nc -zv TARGET_IP 5201

# Verify no firewall blocking
sudo iptables -L | grep 5201
```

2. Performance Problems
```bash
# Test with different window sizes
iperf3 -c TARGET_IP -w 256K

# Check for packet loss with UDP
iperf3 -c TARGET_IP -u -b 100M

# Test with parallel streams
iperf3 -c TARGET_IP -P 4
```

3. Network Path Issues
```bash
# Trace route to server
traceroute TARGET_IP

# Check MTU issues
ping -M do -s 1472 TARGET_IP

# Monitor interface errors
ifconfig | grep -i error
```

## Best Practices
1. Testing Methodology
- Run tests during off-peak hours
- Use consistent test durations
- Test both TCP and UDP
- Monitor both directions
- Document baseline performance

2. Performance Monitoring
- Regular bandwidth tests
- Track latency trends
- Monitor packet loss
- Check jitter values
- Record retransmission rates

3. Result Analysis
- Compare with baselines
- Track performance trends
- Document network changes
- Analyze peak times
- Monitor error rates

4. Documentation
- Record test configurations
- Log network changes
- Document performance issues
- Keep baseline measurements
- Track resolution steps

## Tags
#network #performance #bandwidth #testing #monitoring #troubleshooting