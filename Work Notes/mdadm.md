# mdadm: Comprehensive Guide and Usage

## What is mdadm?

mdadm (multiple devices admin) is a Linux utility used to manage and monitor software RAID (Redundant Array of Independent Disks) devices. It's a powerful tool that allows for the creation, management, and monitoring of RAID arrays without the need for additional hardware RAID controllers.

## Key Features

- Create, assemble, manage, and monitor RAID arrays
- Support for various RAID levels (0, 1, 4, 5, 6, 10)
- Hot-swapping of drives
- Array expansion and level migration
- Automatic array reassembly on boot
- Email notifications for array events

## Basic Syntax

The general syntax for mdadm commands is:

```
mdadm [mode] <raiddevice> [options] <component-devices>
```

## Common Operations

### 1. Creating a RAID Array

To create a RAID 5 array:

```bash
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd
```

### 2. Assembling an Existing Array

To assemble all RAID arrays:

```bash
mdadm --assemble --scan
```

### 3. Stopping an Array

```bash
mdadm --stop /dev/md0
```

### 4. Adding a Spare Drive

```bash
mdadm --add /dev/md0 /dev/sde
```

### 5. Removing a Drive

```bash
mdadm /dev/md0 --fail /dev/sdb --remove /dev/sdb
```

### 6. Checking Array Status

```bash
mdadm --detail /dev/md0
```

## Advanced Usage

### 1. Growing an Array

To add a new drive and grow the array:

```bash
mdadm --grow /dev/md0 --raid-devices=4 --add /dev/sde
```

### 2. Changing RAID Level

To migrate from RAID 5 to RAID 6:

```bash
mdadm --grow /dev/md0 --level=6
```

### 3. Scrubbing an Array

To check for and correct errors:

```bash
echo check > /sys/block/md0/md/sync_action
```

### 4. Setting Up Email Notifications

Edit `/etc/mdadm/mdadm.conf`:

```
MAILADDR your.email@example.com
```

## HPC-Specific Considerations

In HPC environments, mdadm is particularly useful for:

1. High-Performance Storage Solutions
   - Create large, high-performance RAID 0 arrays for temporary scratch space
   ```bash
   mdadm --create /dev/md0 --level=0 --raid-devices=4 /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
   ```

2. Data Redundancy for Critical Storage
   - Implement RAID 6 for important data that requires high redundancy
   ```bash
   mdadm --create /dev/md1 --level=6 --raid-devices=6 /dev/sd[b-g]
   ```

3. Automated Monitoring and Maintenance
   - Set up a cron job to regularly check RAID status:
   ```bash
   echo "0 1 * * * root mdadm --detail --scan | grep -v 'ARRAY' > /tmp/mdadm_status.log" > /etc/cron.d/mdadm_check
   ```

4. Performance Tuning
   - Adjust chunk size for optimal performance:
   ```bash
   mdadm --create /dev/md2 --level=5 --raid-devices=3 --chunk=256 /dev/sd[h-j]
   ```

5. Integration with Job Schedulers
   - Check RAID status before job allocation:
   ```bash
   if ! mdadm --detail /dev/md0 | grep -q "State : clean"; then
       echo "RAID array not in clean state. Aborting job."
       exit 1
   fi
   ```

## Best Practices for HPC Use

1. Regular Status Checks: Implement automated, regular checks of RAID status.

2. Performance Monitoring: Use tools like `iostat` in conjunction with mdadm to monitor RAID performance.

3. Spare Drives: Keep spare drives ready for quick replacement in case of failures.

4. Documentation: Maintain detailed documentation of your RAID configurations.

5. Backup Strategy: Even with RAID, maintain a separate backup strategy for critical data.

6. Testing: Regularly test failure scenarios and recovery procedures.

## Troubleshooting Common Issues

1. Failed Drive
   - Identify the failed drive:
     ```bash
     mdadm --detail /dev/md0
     ```
   - Remove and replace the failed drive:
     ```bash
     mdadm /dev/md0 --remove /dev/sdb
     mdadm /dev/md0 --add /dev/sde
     ```

2. Degraded Array
   - Check the status of a degraded array:
     ```bash
     cat /proc/mdstat
     ```
   - Investigate the cause and take appropriate action (e.g., replace failed drive).

3. Array Won't Assemble
   - Force assembly if necessary (use with caution):
     ```bash
     mdadm --assemble --force /dev/md0 /dev/sd[b-e]
     ```

4. Performance Issues
   - Check the read-ahead setting:
     ```bash
     blockdev --getra /dev/md0
     ```
   - Adjust if necessary:
     ```bash
     blockdev --setra 65536 /dev/md0
     ```

Remember, while mdadm is powerful for managing software RAID, always consider the specific needs and constraints of your HPC environment when designing and managing storage solutions.