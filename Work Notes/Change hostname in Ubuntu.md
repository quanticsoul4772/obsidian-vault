To change the hostname in Ubuntu, you'll need to modify a few files and run some commands. Here's a step-by-step process:

  

1. Set the new hostname temporarily:

  ```

  sudo hostname new-hostname

  ```

  

2. Edit the /etc/hostname file:

  ```

  sudo nano /etc/hostname

  ```

  Replace the existing hostname with your new hostname.

  

3. Update the /etc/hosts file:

  ```

  sudo nano /etc/hosts

  ```

  Find the line with the old hostname and replace it with the new one.

  

4. Restart the hostname service:

  ```

  sudo systemctl restart hostname

  ```

  

5. Reboot your system to ensure all changes take effect:

  ```

  sudo reboot

  ```

  

After the reboot, your system should be using the new hostname. You can verify this by running:

  

```

hostname

```

  

You can change the hostname without rebooting. Here's how to do it:

  

1. Set the new hostname:

  ```

  sudo hostnamectl set-hostname new-hostname

  ```

  This command changes both the transient and static hostname.

  

2. Update the /etc/hosts file:

  ```

  sudo nano /etc/hosts

  ```

  Find the line with the old hostname and replace it with the new one.

  

3. Restart the systemd-hostnamed service:

  ```

  sudo systemctl restart systemd-hostnamed

  ```

  

4. To ensure all running processes are aware of the new hostname, you can run:

  ```

  sudo sysctl kernel.hostname=new-hostname

  ```

  

5. Verify the change:

  ```

  hostnamectl

  ```

  

This method should change your hostname immediately without requiring a reboot. However, some applications might still use the old hostname until they're restarted.

  

A few things to note:

  

- Some services or applications may need to be restarted to recognize the new hostname.

- In some cases, you might need to log out and log back in to see the change in your terminal prompt.

- While this method avoids a full reboot, a reboot is still the most thorough way to ensure all parts of the system are using the new hostname.