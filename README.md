# Roborock with Valetudo: Installing Python 3.6 via Berryconda

**Purpose:**  
This short guide explains how to install Python <=3.6 on Roborock vacuum robots running Valetudo, using Berryconda and BusyBox. It is intended as a reference for others who want to run Python scripts or tools on their robot since I wasted a lot of time. I only tested it on an old Roborock S5. 
This "guide" assumes that you selected to install wget automatically in dustbuilder!

> **Note:** Because there were some errors in the previous commit and I realized that the root/home folders are not persistent, I needed to correct some things.
Since I had to follow my own guide again to get it working after everything got removed I found some more errors. Now everything should be correct!
The way I did things here was not the best for sure. I do not have a lot of experience with such minimal systems. But it worked for me and I wanted to share the process.

**For those who are interested: HW-Info for the S5**
- 4 core ARMv7l CPU
- 512 MB RAM
---

## 1. Login via SSH
Connect to your roborock via SSH using your defined/generated SSH keys as root.

## 1. Install BusyBox for ARMv7l
The distro of the robot is super tiny and extremely limited. Some essential command-line tools (like `bzip2` and `more`) are missing on the Roborock system.  BusyBox provides these in a single binary.

1. Navigate to /mnt/data and create a directory (mine is named feliggs now, you need to adjust your folder name in the following steps too!)
    ```mkdir /mnt/data/feliggs```
    ```cd /mnt/data/feliggs```

3. Download the Berryconda binary from this link: ```wget https://busybox.net/downloads/binaries/1.21.1/busybox-armv7l --no-check-certificate```

4. Make the binary executable: ```chmod +x busybox-armv7l```

5. Create links (inside bin because persistent PATH edit was not possible and bin is already inside the PATH)

    ```ln -sf /mnt/data/feliggs/busybox-armv7l /bin/bzip2```
   
    ```ln -sf /mnt/data/feliggs/busybox-armv7l /bin/more```

## 2. Install Berryconda
Download and run the Berryconda installer for ARMv7l:

    wget https://github.com/jjhelmus/berryconda/releases/download/v2.0.0/Berryconda3-2.0.0-Linux-armv7l.sh
    bash Berryconda3-2.0.0-Linux-armv7l.sh
During the installation process you are asked for the location. Choose your folder again (/mnt/data/feliggs/berryconda3 in this example).
After the installation you can remove the installation script to free up some space:

    rm Berryconda3-2.0.0-Linux-armv7l.sh
    
To easily access conda and pip later on create syslinks again:

    ln -s /mnt/data/feliggs/berryconda3/bin/pip /bin/pip
    ln -s /mnt/data/feliggs/berryconda3/bin/conda /bin/conda
    
## 3. Create and Activate a Conda Environment

    conda create -n your_env python=3.6  
    source activate your_env

## Why do you need Python on your Roborock??

Running Python on your Roborock opens up a wide range of automation and integration possibilities. Beyond typical automation tasks, you can use Python for custom data collection, device integration, and more. For example, I’ve used it to scrape Bitcoin prices using `requests`-and it worked flawlessly.

However, my main motivation for installing Python was to run an MQTT broker directly on the robot using [HBMQTT](https://hbmqtt.readthedocs.io/en/latest/). This allows me to connect the robot with other devices (such as ESP32s) **without needing an extra computer or Raspberry Pi**. Since Valetudo does not provide a built-in MQTT broker, HBMQTT (a Python-based MQTT broker) is a suitable alternative.

## Why HBMQTT?

- **HBMQTT** is an open-source MQTT broker and client, built on Python’s `asyncio` framework, supporting MQTT 3.1.1 with features like QoS 0/1/2, authentication, TCP/websocket/SSL support, and more.
- I tried cross-compiling Mosquitto and NanoMQ, but ran into issues-HBMQTT worked out-of-the-box with Python.
- Although HBMQTT is no longer actively maintained, version 0.6 is stable and meets my requirements.

## HBMQTT Installation Steps

> **Note:** The Roborock’s root filesystem is not persistent. Always install your tools and environments in a persistent location like `/mnt/data`.

### 1. Install Python (Berryconda) in `/mnt/data/feliggs`: (adjust folder name again)
- In the steps above the shown command installs the newest available version 3.6. However for HBMQTT we need version 3.5.2, (python==3.5.2)

    ```conda create -n your_env python=3.5.2```
  
### 2. Install HBMQTT and Required Packages

- Since we created syslinks before we can use pip like usually to install the needed packages. First install HBMQTT and create a syslink again:

    ```pip install hbmqtt==0.6```
      ```ln -s /mnt/data/berryconda3/bin/hbmqtt /bin/hbmqtt```

- When installing hbmqtt like this pip installs incompatible other packages. You need to manually install the following:

    ```pip install websockets==8.1```
    ```pip install pyyaml==5.4.1```

- This allows you to run `hbmqtt` directly from the shell by typing `hbmqtt`. It should work by now :)
- By default, HBMQTT runs without authentication. **Anyone on your network can connect, read, and publish to any topic.**  
- For security, use a configuration file and start the broker with:

    ```hbmqtt -c <your_config_file>```

## Autostart HBMQTT on Boot

Since cron is not available on most Roborock systems, use a persistent boot script:

1. Edit `/etc/rc.local`

```nano /etc/rc.local```
- Add the following line before exit 0:

  ```/bin/hbmqtt > /tmp/hbmqtt.log 2>&1 &```

- Save, exit and reboot. After a reboot, HBMQTT will start automatically-even before Valetudo.

## Performance

- HBMQTT has minimal resource usage. On my setup, all four CPU cores stay below 4% usage, even with Valetudo running.

## Integrating with Valetudo

- In the Valetudo Web UI, go to **Connectivity → MQTT**.
- Use the following settings:
  - **Host:** `localhost`
  - **Port:** `1883`
  - **TLS:** Off
  - **Credentials:** None (unless configured in HBMQTT)
  - **Client Certificate:** None
  - Adjust other settings as needed.

Valetudo will now communicate with your local MQTT broker.

## Notes
- **Berryconda** is no longer actively maintained, but works well for Python 3.6 on ARMv7l devices. 
- **BusyBox** is used here to provide missing utilities required by the installer and Conda.
- For newer Python versions or 64-bit systems, consider [Miniforge](https://github.com/conda-forge/miniforge).
- The ROM memory is small, keep that in mind!
- **Disclaimer:** I take no responsibility for any damage to your device. Software modifications like these can always lead to corruption or malfunction. Proceed at your own risk!

**Enjoy hacking your Roborock :)**


