# High-Performance Computer Diagnostic Tools for Rocky Linux

  

Here's a list of diagnostic tools particularly useful for Rocky Linux systems, along with installation instructions and basic usage:

  

1. memtest86+

  - Installation: `sudo dnf install memtest86+`

  - Usage: Reboot and select memtest86+ from the GRUB menu

  

2. stress-ng

  - Installation: `sudo dnf install stress-ng`

  - Usage: `stress-ng --cpu 8 --io 4 --vm 2 --vm-bytes 128M --timeout 10s`

  

3. nvidia-smi (if you have NVIDIA GPUs)

  - Installation: Typically included with NVIDIA drivers

  - Usage: `nvidia-smi`

  

4. htop

  - Installation: `sudo dnf install htop`

  - Usage: Simply run `htop` in terminal

  

5. iperf3

  - Installation: `sudo dnf install iperf3`

  - Usage: 

   - Server: `iperf3 -s`

   - Client: `iperf3 -c server_ip`

  

6. lm_sensors

  - Installation: `sudo dnf install lm_sensors`

  - Usage: 

   - Configure: `sudo sensors-detect`

   - Run: `sensors`

  

7. smartmontools

  - Installation: `sudo dnf install smartmontools`

  - Usage: `sudo smartctl -a /dev/sda` (replace sda with your drive)

  

8. ipmitool (if your hardware supports IPMI)

  - Installation: `sudo dnf install ipmitool`

  - Usage: `ipmitool sensor list`

  

9. mdadm (for RAID management)

  - Installation: `sudo dnf install mdadm`

  - Usage: `sudo mdadm --detail /dev/md0` (replace md0 with your RAID device)

  

10. dstat

  - Installation: `sudo dnf install dstat`

  - Usage: `dstat`

  

11. sysstat (includes sar)

  - Installation: `sudo dnf install sysstat`

  - Usage: 

   - Enable service: `sudo systemctl enable --now sysstat`

   - Run: `sar -u 1 10` (CPU usage, 1 second intervals, 10 times)

  

12. nmon

  - Installation: `sudo dnf install nmon`

  - Usage: Simply run `nmon` in terminal

  

13. hdparm

  - Installation: `sudo dnf install hdparm`

  - Usage: `sudo hdparm -tT /dev/sda` (replace sda with your drive)

  

14. fio (Flexible I/O Tester)

  - Installation: `sudo dnf install fio`

  - Usage: `fio --name=random-write --ioengine=posixaio --rw=randwrite --bs=4k --size=4g --numjobs=1 --runtime=60 --time_based --end_fsync=1`

  

15. perf (Linux profiling with performance counters)

  - Installation: `sudo dnf install perf`

  - Usage: `sudo perf stat -d program_name`

  

16. tuned (system tuning daemon)

  - Installation: `sudo dnf install tuned`

  - Usage:

   - Start service: `sudo systemctl enable --now tuned`

   - Set profile: `sudo tuned-adm profile throughput-performance`

  

17. numactl (NUMA control)

  - Installation: `sudo dnf install numactl`

  - Usage: `numactl --hardware`

  

18. hwloc (Portable Hardware Locality)

  - Installation: `sudo dnf install hwloc`

  - Usage: `lstopo`

  

Most of these tools are available in the standard Rocky Linux repositories and can be installed using `dnf`. Tools like `top`, `free`, and `lspci` typically come pre-installed on Rocky Linux systems.

  

Remember to use `sudo` when necessary, especially for tools that require system-level access. Always be cautious when running stress tests or making changes to hardware settings.

  

Additional Notes for Rocky Linux:

1. You may need to enable additional repositories like EPEL (Extra Packages for Enterprise Linux) for some packages:

  ```

  sudo dnf install epel-release

  sudo dnf update

  ```

2. For NVIDIA drivers and CUDA tools, you might need to add the NVIDIA repository:

  ```

  sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo

  sudo dnf clean all

  sudo dnf module install nvidia-driver:latest-dkms

  ```

3. Rocky Linux, being a RHEL derivative, is designed for stability. Always test new tools in a non-production environment first.