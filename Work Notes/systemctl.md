# Understanding and Utilizing systemctl

## What is systemctl?

systemctl is the central management tool for controlling the systemd system and service manager. It's used to examine and control the state of systemd system and service manager.

## Key Concepts

1. **Units**: The objects that systemd knows how to manage. These include:
   - Services (.service)
   - Mount points (.mount)
   - Devices (.device)
   - Sockets (.socket)

2. **Targets**: Groups of units (similar to runlevels in SysV init systems)

## Basic systemctl Commands

1. Check the status of a service:
   ```
   systemctl status service_name.service
   ```

2. Start a service:
   ```
   sudo systemctl start service_name.service
   ```

3. Stop a service:
   ```
   sudo systemctl stop service_name.service
   ```

4. Restart a service:
   ```
   sudo systemctl restart service_name.service
   ```

5. Enable a service to start at boot:
   ```
   sudo systemctl enable service_name.service
   ```

6. Disable a service from starting at boot:
   ```
   sudo systemctl disable service_name.service
   ```

7. Check if a service is enabled:
   ```
   systemctl is-enabled service_name.service
   ```

8. Reload a service's configuration:
   ```
   sudo systemctl reload service_name.service
   ```

## Working with System State

1. Reboot the system:
   ```
   sudo systemctl reboot
   ```

2. Shut down the system:
   ```
   sudo systemctl poweroff
   ```

3. Put the system in suspend mode:
   ```
   sudo systemctl suspend
   ```

## Viewing System Information

1. List all running units:
   ```
   systemctl list-units
   ```

2. List all failed units:
   ```
   systemctl --failed
   ```

3. List all installed unit files:
   ```
   systemctl list-unit-files
   ```

4. Show the default target:
   ```
   systemctl get-default
   ```

## Working with Targets

1. Change the current target:
   ```
   sudo systemctl isolate target_name.target
   ```

2. Set the default target:
   ```
   sudo systemctl set-default target_name.target
   ```

## Analyzing Boot Performance

1. Show system boot-up performance:
   ```
   systemd-analyze
   ```

2. Show unit initialization times:
   ```
   systemd-analyze blame
   ```

## Masking and Unmasking Units

1. Mask a unit (to prevent it from being started):
   ```
   sudo systemctl mask unit_name.service
   ```

2. Unmask a unit:
   ```
   sudo systemctl unmask unit_name.service
   ```

## Tips for Using systemctl

1. You can often omit the `.service` suffix for service units.
2. Use tab completion to find unit names quickly.
3. The `--now` option can be used with enable/disable to start/stop the service immediately.
4. Use `systemctl edit unit_name.service` to create override files for units.

Remember, many systemctl commands require root privileges, so you'll often need to use `sudo`.