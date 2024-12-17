# Maintaining NVIDIA Drivers in Rocky Linux

## Overview
- Task Category: Driver Management
- Systems Affected: Rocky Linux with NVIDIA GPUs
- Prerequisites: sudo access, compatible NVIDIA GPU

## Content

1. Determining Appropriate Driver:
- Check NVIDIA's website for compatible driver
- Identify GPU:
```bash
lspci | grep -i nvidia
```

2. System Preparation
```bash
sudo dnf update
sudo dnf install kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig
```

3. Installation Methods
### Using EPEL and RPM Fusion (Recommended)
```bash
# Enable EPEL
sudo dnf install epel-release

# Enable RPM Fusion
sudo dnf install https://download1.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-$(rpm -E %rhel).noarch.rpm

# Install NVIDIA driver
sudo dnf install akmod-nvidia

# For CUDA support
sudo dnf install xorg-x11-drv-nvidia-cuda
```

[Rest of the content remains the same...]

## Related Documentation
- [[SysAdmin - Maintaining NVIDIA Ubuntu]] - Similar guide for Ubuntu
- [[SysAdmin - NVIDIA Driver Update Script]] - Automation script
- [[Tools - nvidia-smi Guide]] - GPU monitoring tool guide
- [[Doc - SELinux Configuration]] - SELinux considerations

#sysadmin #nvidia #rocky-linux