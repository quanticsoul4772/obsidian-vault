Here's a list of best practices for installing, updating, and maintaining NVIDIA drivers in Rocky Linux:

  

1. Determine the appropriate driver:

  - Check NVIDIA's website for the latest driver compatible with your GPU.

  - Use the following command to identify your GPU:

   ```

   lspci | grep -i nvidia

   ```

  

2. Prepare the system:

  - Ensure your system is up to date:

   ```

   sudo dnf update

   ```

  - Install necessary packages:

   ```

   sudo dnf install kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig

   ```

  

3. Installation methods:

  a. Using EPEL and RPM Fusion repositories (recommended):

   - Enable EPEL repository:

    ```

    sudo dnf install epel-release

    ```

   - Enable RPM Fusion repositories:

    ```

    sudo dnf install https://download1.rpmfusion.org/free/el/rpmfusion-free-release-$(rpm -E %rhel).noarch.rpm

    sudo dnf install https://download1.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-$(rpm -E %rhel).noarch.rpm

    ```

   - Install NVIDIA driver:

    ```

    sudo dnf install akmod-nvidia

    ```

   - For CUDA support:

    ```

    sudo dnf install xorg-x11-drv-nvidia-cuda

    ```

  

  b. Using NVIDIA's official .run file (for advanced users):

   - Download from NVIDIA's website

   - Stop the display manager:

    ```

    sudo systemctl isolate multi-user.target

    ```

   - Run the installer:

    ```

    sudo sh ./NVIDIA-Linux-x86_64-xxx.xx.run

    ```

  

4. Post-installation verification:

  - Check driver version:

   ```

   nvidia-smi

   ```

  - Verify OpenGL:

   ```

   glxinfo | grep "OpenGL version"

   ```

  

5. Updating drivers:

  - For drivers installed via repositories:

   ```

   sudo dnf update

   ```

  - For manually installed drivers, repeat the installation process with the new driver version

  

6. Kernel updates:

  - If using repository drivers (akmod-nvidia), they should automatically rebuild for new kernels

  - For manual installations, you may need to reinstall the driver after major kernel updates

  

7. Troubleshooting:

  - Keep old kernels installed for fallback

  - Use nomodeset kernel parameter if you encounter boot issues

  - Be prepared to use the command line if GUI fails to start

  

8. Maintenance best practices:

  - Regularly check for driver updates (every few months)

  - Monitor GPU performance and temperatures using nvidia-smi

  - Keep your system updated:

   ```

   sudo dnf update

   ```

  

9. CUDA toolkit (if needed):

  - Install CUDA separately from the driver

  - Use CUDA runfile for installation if you need a specific version

  

10. Secure Boot:

  - If Secure Boot is enabled, you'll need to sign the NVIDIA modules

  - Consider using repository drivers which handle this automatically

  

11. Uninstallation (if needed):

  - For repository drivers:

   ```

   sudo dnf remove "*nvidia*" "*cuda*"

   ```

  - For manual installations:

   ```

   sudo nvidia-uninstall

   ```

  

12. Multiple GPUs:

  - Ensure proper power and cooling

  - Use nvidia-smi to monitor all GPUs

  

13. Wayland vs X11:

  - NVIDIA drivers work best with X11

  - If using Wayland, be prepared for potential issues

  

14. Backup:

  - Always backup important data before major driver changes

  - Consider creating a system snapshot if possible

  

15. Testing:

  - After installation or updates, test GPU-intensive applications

  - Run benchmarks to ensure performance is as expected

  

16. SELinux considerations:

  - Rocky Linux uses SELinux by default. Ensure it's not blocking NVIDIA driver functionality

  - Use `sudo ausearch -m avc -ts recent` to check for any SELinux-related blocks

  

17. Firewall settings:

  - If you're using the GPU for compute tasks over the network, ensure necessary ports are open

  

18. Documentation:

  - Keep notes on which driver versions work best for your system

  - Document any specific tweaks or configurations you've made

  

Remember, Rocky Linux aims for stability, so it's often better to stick with the versions provided through official repositories unless you have specific needs that require the latest drivers.