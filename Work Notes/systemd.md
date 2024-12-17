Systemd Overview:

Systemd is a comprehensive system and service manager for Linux. It's designed to be backwards compatible with SysV init scripts, and provides extensive features for starting, stopping, and managing services.

  

Key Components:

  

1. systemd (core): 

  - The main system and service manager

  - PID 1 process that bootstraps the user space and manages user processes

  

2. systemctl:

  - Command-line tool for controlling and managing systemd and services

  

3. journald:

  - Collects and manages system logs

  

4. logind:

  - Manages user logins and sessions

  

5. networkd:

  - Network configuration manager

  

6. resolved:

  - Network name resolution manager

  

7. timesyncd:

  - NTP client for time synchronization

  

Key Features:

  

1. Socket and Bus Activation:

  - Services can be started on-demand when needed

  

2. Snapshot and Restore:

  - Can save and restore system state

  

3. Transaction-based Dependencies:

  - Ensures services start in the correct order

  

4. Tracking Processes:

  - Uses cgroups to accurately track processes

  

5. Performance Optimization:

  - Aggressive parallelization of startup

  

6. Mount and Automount Point Management:

  - Handles filesystem mounts dynamically

  

7. Logging:

  - Provides a centralized, indexed logging system

  

Unit Files:

Systemd uses "unit files" for configuration. These are typically stored in /etc/systemd/system/ or /usr/lib/systemd/system/. Common types include:

  

1. .service: Define services

2. .socket: Define sockets for socket activation

3. .device: Define device units

4. .mount: Define mount points

5. .target: Define targets (groups of units)

  

Example of a simple .service file:

  

```

[Unit]

Description=My Custom Service

After=network.target

  

[Service]

ExecStart=/usr/bin/my_custom_script.sh

Restart=on-failure

  

[Install]

WantedBy=multi-user.target

```

  

Common systemctl Commands:

  

1. Start a service:

  ```

  sudo systemctl start service_name.service

  ```

  

2. Stop a service:

  ```

  sudo systemctl stop service_name.service

  ```

  

3. Restart a service:

  ```

  sudo systemctl restart service_name.service

  ```

  

4. Enable a service to start on boot:

  ```

  sudo systemctl enable service_name.service

  ```

  

5. Check status of a service:

  ```

  systemctl status service_name.service

  ```

  

6. List all loaded units:

  ```

  systemctl list-units

  ```

  

7. List all installed unit files:

  ```

  systemctl list-unit-files

  ```

  

Targets:

Systemd uses targets instead of runlevels. Key targets include:

  

- poweroff.target

- rescue.target

- multi-user.target

- graphical.target

- reboot.target

  

To change targets:

```

sudo systemctl isolate target_name.target

```

  

Logging with journalctl:

Systemd provides a powerful logging system. Some useful commands:

  

1. View all logs:

  ```

  journalctl

  ```

  

2. View logs for a specific service:

  ```

  journalctl -u service_name.service

  ```

  

3. View logs since last boot:

  ```

  journalctl -b

  ```

  

Systemd has faced some controversy due to its complexity and departure from Unix philosophy, but it has been widely adopted for its powerful features and efficiency improvements.

  

Would you like me to elaborate on any specific aspect of systemd, such as its boot process, service management, or logging capabilities?

  

### systemd

Let's dive deeper into systemd and some key aspects of init systems in general.

  

Systemd:

Systemd is a modern init system and system manager that has become the de facto standard for many Linux distributions. It was designed to overcome limitations of the traditional SysV init system.

  

Key features of systemd:

  

1. Parallelization: 

  Systemd can start services in parallel, significantly reducing boot time.

  

2. On-demand service activation:

  Services can be started when needed rather than at boot time.

  

3. Dependency management:

  Systemd handles complex dependencies between services automatically.

  

4. Unified configuration:

  Uses simple, standardized unit files for configuration.

  

5. System state tracking:

  Keeps track of processes, making it easier to manage and monitor the system.

  

6. Journal:

  Provides a centralized logging system (journald) that collects and manages log data.

  

7. Targets:

  Replaces runlevels with more flexible "target" units.

  

Core concepts in systemd:

  

1. Units: 

  The basic building blocks in systemd. Types include:

  - service: for running daemons

  - socket: for socket-based activation

  - target: for grouping units

  - mount: for managing mount points

  

2. Unit files:

  Configuration files that define how systemd should manage a unit.

  

3. Targets:

  Used to bring the system to a specific state, similar to runlevels.

  

Key systemd commands:

  

- `systemctl start/stop/restart/status <unit>`

- `systemctl enable/disable <unit>`

- `journalctl`: for viewing logs

  

Specific aspects of init systems:

  

1. Process management:

  Init systems are responsible for starting, stopping, and monitoring system processes.

  

2. Boot process:

  They manage the boot sequence, bringing the system from power-on to a usable state.

  

3. Service management:

  Init systems handle starting and stopping services, often based on system state or dependencies.

  

4. System state:

  They define and manage different system states (like single-user or multi-user modes).

  

5. Shutdown and reboot:

  Init systems are responsible for gracefully shutting down or rebooting the system.

  

6. Orphan process adoption:

  As PID 1, init systems adopt orphaned processes, preventing zombie processes.

  

Differences between init systems:

  

1. SysV init: 

  - Uses shell scripts

  - Sequential startup

  - Runlevel-based

  

2. Upstart:

  - Event-driven

  - Parallel startup

  - Backwards compatible with SysV init scripts

  

3. Systemd:

  - Aggressive parallelization

  - Dependency-based

  - Unified configuration approach