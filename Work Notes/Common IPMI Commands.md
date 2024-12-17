
Here's a list of frequently used IPMI commands you can run with `ipmitool`:
  

1. Power Control:

  - Power status: `ipmitool power status`

  - Power on: `ipmitool power on`

  - Power off: `ipmitool power off`

  - Power cycle: `ipmitool power cycle`

  - Soft shutdown: `ipmitool power soft`

  

2. System Information:

  - Basic system info: `ipmitool fru print`

  - Detailed system info: `ipmitool mc info`

  

3. Sensor Readings:

  - All sensor readings: `ipmitool sensor list`

  - Specific sensor reading: `ipmitool sensor get "CPU Temp"`

  

4. System Event Log (SEL):

  - View SEL: `ipmitool sel list`

  - Clear SEL: `ipmitool sel clear`

  

5. Chassis Control:

  - Chassis status: `ipmitool chassis status`

  - Identify LED on: `ipmitool chassis identify 30` # 30 seconds

  

6. Boot Device Control:

  - Set next boot to PXE: `ipmitool chassis bootdev pxe`

  - Set next boot to disk: `ipmitool chassis bootdev disk`

  

7. User Management:

  - List users: `ipmitool user list 1`

  - Set user password: `ipmitool user set password 2 mypassword`

  

8. Network Configuration:

  - Show network settings: `ipmitool lan print 1`

  - Set static IP: `ipmitool lan set 1 ipaddr 192.168.1.100`

  

9. SOL (Serial Over LAN):

  - Activate SOL: `ipmitool sol activate`

  - Deactivate SOL: `ipmitool sol deactivate`

  

10. Firmware Management:

  - Get firmware version: `ipmitool mc info`

  

11. Raw Commands (for advanced users):

  - Example: `ipmitool raw 0x00 0x01`

  

Notes:

- Replace `ipmitool` with `sudo ipmitool` if you need root privileges.

- For remote management, add `-H [IP] -U [username] -P [password]` before the command.

- Example of a remote command:

 ```

 ipmitool -H 192.168.1.100 -U admin -P password power status

 ```

- Some commands may vary slightly depending on your specific hardware.

  

### ipmitool

IPMI (Intelligent Platform Management Interface) is a standardized computer system interface used for out-of-band management of computer systems and monitoring their operation. ipmitool is a utility for managing and configuring devices that support the IPMI protocol. It's particularly useful for server management, especially in data centers or remote locations.

  

Here's a detailed explanation of how to use ipmitool:

  

Basic Syntax:

```

ipmitool [options] <command>

```

  

Common Options:

  1. -I <interface>: Specify which interface to use (lan, lanplus, open, etc.)

2. -H <hostname>: Remote host name or IP address

3. -U <username>: Remote username

4. -P <password>: Remote password

5. -L <privilege_level>: Remote session privilege level

6. -p <port>: Remote RMCP port (default is 623)

  

Key Commands and Use Cases:

  

1. Power Management:

  - Power status: 

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> power status

   ```

  - Power on:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> power on

   ```

  - Power off:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> power off

   ```

  - Power cycle:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> power cycle

   ```

  

2. Sensor Information:

  - List all sensors:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> sensor list

   ```

  - Get specific sensor reading:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> sensor get "CPU Temp"

   ```

  

3. System Event Log (SEL):

  - View SEL:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> sel list

   ```

  - Clear SEL:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> sel clear

   ```

  

4. Chassis Information:

  - Get chassis status:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> chassis status

   ```

  - Get chassis power policy:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> chassis policy list

   ```

  

5. BMC (Baseboard Management Controller) Information:

  - Get BMC info:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> bmc info

   ```

  - Reset BMC:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> bmc reset cold

   ```

  

6. User Management:

  - List users:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> user list

   ```

  - Set user password:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> user set password 2 newpassword

   ```

  

7. Network Configuration:

  - Get LAN configuration:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> lan print

   ```

  - Set static IP:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> lan set 1 ipaddr 192.168.1.100

   ```

  

8. Serial-over-LAN (SOL):

  - Activate SOL:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> sol activate

   ```

  - Deactivate SOL:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> sol deactivate

   ```

  

9. Firmware Management:

  - Get firmware version:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> mc info

   ```

  

10. System Boot Options:

  - Set next boot to PXE:

   ```

   ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> chassis bootdev pxe

   ```

  

Advanced Usage:

  

1. Raw Commands:

  Send raw IPMI commands when standard commands don't suffice:

  ```

  ipmitool -I lanplus -H <IP> -U <USER> -P <PASS> raw 0x00 0x01

  ```

  

2. Using with Local System:

  If running on the local system with IPMI support:

  ```

  ipmitool <command>

  ```

  

3. Scripting:

  ipmitool can be used in scripts for automated management:

  ```bash

  #!/bin/bash

  IPMI_HOSTS="192.168.1.100 192.168.1.101 192.168.1.102"

  for HOST in $IPMI_HOSTS; do

    ipmitool -I lanplus -H $HOST -U admin -P password power status

  done

  ```
 

Important Notes:

1. Always use secure passwords and consider using a configuration file to store credentials securely.

2. Some commands may require administrator privileges.

3. The exact commands and features available may vary depending on the server hardware and IPMI implementation.

4. Be cautious with power management commands, especially in production environments.

5. For security, use the lanplus interface when possible, as it supports encryption.