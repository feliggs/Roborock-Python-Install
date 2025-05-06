# Roborock with Valetudo: Installing Python 3.6 via Berryconda

**Purpose:**  
This short guide explains how to install Python 3.6 on Roborock vacuum robots running Valetudo, using Berryconda and BusyBox. It is intended as a reference for others who want to run Python scripts or tools on their robot since I wasted a lot of time. I only tested it on an old Roborock S5. 

This "guide" assumes that you selected to install wget automatically in dustbuilder!

**HW-Info**
- 4 core ARMv7l CPU
- 512 MB RAM
---

## 1. Login via SSH
Connect to your roborock via SSH using your defined/generated SSH keys as root.

## 1. Install BusyBox for ARMv7l
The distro of the robot is super tiny and extremely limited. Some essential command-line tools (like `bzip2` and `more`) are missing on the Roborock system.  BusyBox provides these in a single binary.

1. Download binary from this link: ```wget [https://busybox.net/downloads/binaries/1.21.1/busybox-armv7l](https://busybox.net/downloads/binaries/1.21.1/busybox-armv7l) --no-check-certificate```

2. Make the binary executable: ```chmod +x busybox-armv7l```

4. Create links

    ```ln -sf /root/busybox-armv7l /root/bzip2  ```
   
    ```ln -sf /root/busybox-armv7l /root/more```

6. Add the directory to your PATH

    ```export PATH="/root:$PATH"```

## 2. Install Berryconda
Download and run the Berryconda installer for ARMv7l: (choices during install: yes, blank, yes)

    wget [https://github.com/jjhelmus/berryconda/releases/download/v2.0.0/Berryconda3-2.0.0-Linux-armv7l.sh](https://github.com/jjhelmus/berryconda/releases/download/v2.0.0/Berryconda3-2.0.0-Linux-armv7l.sh)  
    bash Berryconda3-2.0.0-Linux-armv7l.sh

## 3. Create and Activate a Conda Environment

    conda create -n your_env python=3.6  
    source activate your_env

## Notes
- **Berryconda** is no longer actively maintained, but works well for Python 3.6 on ARMv7l devices. 
- **BusyBox** is used here to provide missing utilities required by the installer and Conda.
- For newer Python versions or 64-bit systems, consider [Miniforge](https://github.com/conda-forge/miniforge).
- The ROM memory is small, keep that in mind!
- **Disclaimer:** I take no responsibility for any damage to your device. Software modifications like these can always lead to corruption or malfunction. Proceed at your own risk!

**Enjoy hacking your Roborock :)**


