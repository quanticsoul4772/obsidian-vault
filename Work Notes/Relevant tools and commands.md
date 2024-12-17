
### Using `nvidia-smi`

`nvidia-smi` is a command-line utility for monitoring and managing NVIDIA GPUs. Here are some common and useful commands:
  

#### Basic Commands

1. **Check GPU Status**

  ```sh

  nvidia-smi

  ```

  This command provides a summary of the GPUs connected to the system, including memory usage, temperature, and power usage.

  

2. **List All GPUs**

  ```sh

  nvidia-smi -L

  ```

  This command lists all the GPUs connected to the system.

  

3. **Monitor GPU Utilization**

  ```sh

  watch -n 1 nvidia-smi

  ```

  This command refreshes the output every second to monitor GPU utilization in real-time.

  

#### Query Specific Information

1. **Detailed GPU Information**

  ```sh

  nvidia-smi -q

  ```

  Displays detailed information about each GPU.

  

2. **Query Specific Attributes**

  ```sh

  nvidia-smi --query-gpu=memory.total,memory.used,memory.free,utilization.gpu --format=csv

  ```

  This command queries specific attributes and displays them in CSV format.

  

3. **Display GPU Processes**

  ```sh

  nvidia-smi pmon -i 0

  ```

  Displays information about the processes running on GPU 0.

  

#### Device Modification

1. **Set Persistence Mode**

  ```sh

  sudo nvidia-smi -pm 1

  ```

  Enables persistence mode.

  

2. **Set Power Limit**

  ```sh

  sudo nvidia-smi -pl 100

  ```

  Sets the power limit of the GPU to 100 watts.

  

3. **Reset GPU**

  ```sh

  sudo nvidia-smi --gpu-reset -i 0

  ```

  Resets GPU 0.

  

### Disk Management with `lsblk` and `fdisk`

1. **List Block Devices**

  ```sh

  lsblk

  ```

  Lists information about all available or specified block devices.

  

2. **Partitioning with `fdisk`**

  ```sh

  sudo fdisk /dev/sdX

  ```

  - Inside `fdisk`, you can use:

   - `p`: Print the partition table.

   - `n`: Add a new partition.

   - `d`: Delete a partition.

   - `w`: Write the table to disk and exit.

   - `q`: Quit without saving changes.

  

3. **Formatting a Partition**

  ```sh

  sudo mkfs.ext4 /dev/sdX1

  ```

  Formats the partition `/dev/sdX1` with the ext4 filesystem.

  

4. **Mounting a Partition**

  ```sh

  sudo mkdir /mnt/data

  sudo mount /dev/sdX1 /mnt/data

  ```

  

### Additional Commands for System Information

1. **Check Disk Usage**

  ```sh

  df -h

  ```

  

2. **System Information**

  ```sh

  uname -a

  ```

  

3. **Check Memory Usage**

  ```sh

  free -h

  ```

  

### Practice and Preparation

- **Virtual Environment**: Set up a virtual machine with Ubuntu to practice these commands.

- **Follow Tutorials**: Use online tutorials to get step-by-step instructions on partitioning and GPU management.

- **Create Scenarios**: Simulate real-world scenarios like partitioning a new drive, setting up a GPU, and monitoring system resources.
