# Maintaining NVIDIA Drivers in Ubuntu

## Overview
- Task Category: Driver Management
- Systems Affected: Ubuntu Linux with NVIDIA GPUs
- Prerequisites: sudo access, compatible NVIDIA GPU

## Content

1. Determining Appropriate Driver:
- Use `ubuntu-drivers devices` to see recommended drivers
- Check NVIDIA's website for latest compatible driver

2. Installation Methods
### Ubuntu's Built-in Additional Drivers Tool (Recommended)
- Open "Software & Updates"
- Go to "Additional Drivers" tab
- Select recommended NVIDIA driver
- Apply changes and reboot

### Ubuntu's Command-line Tool
```bash
sudo ubuntu-drivers autoinstall
```

### NVIDIA's Official .run File (Advanced)
```bash
sudo systemctl stop gdm  # (or lightdm)
sudo sh ./NVIDIA-Linux-x86_64-xxx.xx.run
```

[Rest of the content remains the same...]

## Related Documentation
- [[SysAdmin - Maintaining NVIDIA Rocky]] - Similar guide for Rocky Linux
- [[SysAdmin - NVIDIA Driver Update Script]] - Automation script
- [[Tools - nvidia-smi Guide]] - GPU monitoring tool guide

#sysadmin #nvidia #ubuntu