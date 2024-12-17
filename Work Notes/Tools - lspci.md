# PCI Device Management with lspci

## Overview
lspci is a utility for displaying information about PCI buses in the system and devices connected to them. It's crucial for hardware diagnostics and system inspection.

## Basic Usage

### Information Display
```bash
# Basic device listing
lspci

# Verbose output
lspci -v

# Very verbose output
lspci -vv

# Most detailed information
lspci -vvv

# Show numeric IDs
lspci -n

# Show kernel drivers
lspci -k
```

### Specific Device Information
```bash
# Get information about a specific device
lspci -s 00:02.0 -v

# Show device tree
lspci -t

# Show PCI capabilities
lspci -vv | grep -A10 "Capabilities"
```

## Automation Scripts

### Device Inventory
```bash
#!/bin/bash
# Generate PCI device inventory

create_inventory() {
    local output_file="pci_inventory_$(date +%Y%m%d).json"
    
    echo "{" > "$output_file"
    echo "  \"devices\": [" >> "$output_file"
    
    lspci -vmm | awk '
    BEGIN { ORS=""; first=1 }
    /^$/ { next }
    /^Device:/ {
        if (!first) print ","
        first=0
        print "\n    {"
        flag=1
    }
    !/^$/ {
        if (flag) {
            printf "\n      \"%s\": \"%s\"", $1, substr($0, index($0,$2))
            flag=0
        }
    }
    END { print "\n  ]" }
    ' >> "$output_file"
    
    echo "}" >> "$output_file"
}
```

### GPU Detection
```bash
#!/bin/bash
# Detect and report GPU information

check_gpus() {
    local report_file="gpu_report.txt"
    echo "GPU Report - $(date)" > "$report_file"
    echo "===================" >> "$report_file"
    
    # Find all GPU devices
    lspci | grep -i 'vga\|3d\|display' > gpu_list.tmp
    
    while read -r line; do
        device_id=$(echo "$line" | cut -d' ' -f1)
        echo "Device: $line" >> "$report_file"
        echo "Detailed Information:" >> "$report_file"
        lspci -v -s "$device_id" >> "$report_file"
        echo "-------------------" >> "$report_file"
    done < gpu_list.tmp
    
    rm gpu_list.tmp
}
```

### Network Card Analysis
```bash
#!/bin/bash
# Analyze network interfaces

check_network_cards() {
    local output_dir="network_analysis"
    mkdir -p "$output_dir"
    
    # Find network controllers
    lspci | grep -i ethernet > "$output_dir/network_cards.txt"
    
    # Get detailed information
    while read -r line; do
        device_id=$(echo "$line" | cut -d' ' -f1)
        lspci -vv -s "$device_id" > "$output_dir/${device_id}_details.txt"
        
        # Check kernel driver
        lspci -k -s "$device_id" > "$output_dir/${device_id}_driver.txt"
    done < "$output_dir/network_cards.txt"
}
```

## Device Monitoring

### Performance Issues Detection
```bash
#!/bin/bash
# Monitor PCI device errors

monitor_pci_errors() {
    local log_file="/var/log/pci_errors.log"
    
    echo "PCI Error Check - $(date)" >> "$log_file"
    
    # Check for correctable errors
    lspci -vv | grep -A5 "Express Root Port" | grep "Corrected Errors" >> "$log_file"
    
    # Check for uncorrectable errors
    lspci -vv | grep -A5 "Express Root Port" | grep "Uncorrected Errors" >> "$log_file"
}
```

### Hardware Changes Monitor
```bash
#!/bin/bash
# Track PCI device changes

track_hardware_changes() {
    local baseline_file="/var/lib/hardware/pci_baseline.txt"
    local current_file="/var/lib/hardware/pci_current.txt"
    local changes_file="/var/lib/hardware/pci_changes.log"
    
    mkdir -p "$(dirname "$baseline_file")"
    
    # Create current snapshot
    lspci -nn > "$current_file"
    
    # Compare with baseline if it exists
    if [ -f "$baseline_file" ]; then
        diff "$baseline_file" "$current_file" > "$changes_file"
    else
        cp "$current_file" "$baseline_file"
        echo "Initial baseline created" > "$changes_file"
    fi
}
```

## Integration Examples

### Prometheus Integration
```bash
#!/bin/bash
# Generate PCI metrics for Prometheus

generate_pci_metrics() {
    echo "# HELP pci_device_count Number of PCI devices by type"
    echo "# TYPE pci_device_count gauge"
    
    # Count devices by type
    lspci | awk -F: '{print $2}' | sort | uniq -c | while read count type; do
        type_clean=$(echo "$type" | tr -d ' ' | tr '[:upper:]' '[:lower:]' | tr ' ' '_')
        echo "pci_device_count{type=\"$type_clean\"} $count"
    done
}

# Save to node_exporter directory
generate_pci_metrics > /var/lib/node_exporter/pci_metrics.prom
```

### System Information Export
```bash
#!/bin/bash
# Export PCI information in various formats

export_pci_info() {
    local format=$1  # json, xml, or csv
    local output_dir="/var/lib/hardware/exports"
    
    mkdir -p "$output_dir"
    
    case "$format" in
        "json")
            create_inventory  # Uses function from earlier
            ;;
        "xml")
            lspci -x > "$output_dir/pci_info.xml"
            ;;
        "csv")
            lspci -vmm | awk -F':' '
            BEGIN {OFS=","; print "Slot,Device,Class,Vendor,Device_ID,Rev"}
            {gsub(/ /, "", $2); if (NF) print $2}
            ' > "$output_dir/pci_info.csv"
            ;;
    esac
}
```

## Troubleshooting Guide

### Common Issues
1. Device Not Detected
```bash
# Check if device is physically present
lspci -nn

# Check kernel modules
lspci -k

# Check device capabilities
lspci -vv
```

2. Driver Issues
```bash
# Check current driver
lspci -k | grep -A3 "Device"

# Check kernel messages
dmesg | grep pci
```

3. Performance Issues
```bash
# Check link status
lspci -vv | grep -A10 "Status"

# Check power management
lspci -vv | grep -A5 "Power"
```

### Debug Information Collection
```bash
#!/bin/bash
# Collect debug information

collect_debug_info() {
    local debug_dir="pci_debug_$(date +%Y%m%d_%H%M%S)"
    mkdir -p "$debug_dir"
    
    # Basic information
    lspci > "$debug_dir/basic_info.txt"
    
    # Detailed information
    lspci -vv > "$debug_dir/detailed_info.txt"
    
    # Kernel modules
    lspci -k > "$debug_dir/kernel_modules.txt"
    
    # Device tree
    lspci -t > "$debug_dir/device_tree.txt"
    
    # Kernel messages
    dmesg | grep -i pci > "$debug_dir/kernel_messages.txt"
    
    # Create tar archive
    tar -czf "${debug_dir}.tar.gz" "$debug_dir"
    rm -rf "$debug_dir"
}
```

## Tags
#pci #hardware #diagnostics #system-management