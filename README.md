# AIS Catcher Installation Guide

This guide provides step-by-step instructions on how to install AIS Catcher on your Raspberry Pi.

## Features

- Clones the AIS Catcher source code from [https://github.com/jvde-github/AIS-catcher](https://github.com/jvde-github/AIS-catcher)
- Builds the Linux executable from the source code and installs it in the `/usr/local/bin/` folder
- Creates a Systemd service to automatically start AIS Catcher when the Raspberry Pi boots and run it in the background
- Provides Systemd commands to start, stop, restart, and display the status and journalctl logs of AIS Catcher

## Installation

To install AIS Catcher, follow these steps:

1. Copy and paste the following command into your SSH console and press Enter:

```bash
sudo bash -c "$(wget -O - https://raw.githubusercontent.com/abcd567a/install-aiscatcher/master/install-aiscatcher.sh)"
```

2. After the installation is completed, perform the following steps:

   - If you have installed AIS Dispatcher or OpenCPN on your Raspberry Pi, configure them to use UDP Port 10110 and IP 127.0.0.1 or 0.0.0.0.
   - Open the `aiscatcher.conf` file using the following command:

     ```bash
     sudo nano /usr/share/aiscatcher/aiscatcher.conf
     ```

   - Modify the content of the `aiscatcher.conf` file according to your requirements. The default content is shown below:

     ```bash
     -d 00000162
     -v 10
     -M DT
     -gr TUNER 38.6 RTLAGC off
     -s 2304k
     -p 3
     -o 4
     -u 127.0.0.1 10110
     -N 8383
     -N PLUGIN_DIR /usr/share/aiscatcher/my-plugins
     ```

   - Save the file and exit the editor.

3. Reboot your Raspberry Pi.

4. Access the web interface (map, etc.) at `10.0.0.100:8383` (replace `10.0.0.100` with the IP address of your Raspberry Pi).

## PPM Correction

To determine the PPM correction value for the `aiscatcher.conf` file, follow these steps:

1. Install the test software by running the following command:

```bash
sudo apt install rtl-sdr
```

2. Determine the device index of the AIS dongle by running the following command:

```bash
rtl_test -t
```

   The device index will be 0, 1, 2, etc.

3. Use the following command to determine the PPM correction value:

```bash
rtl_test -d n -p60
```

   Replace `n` with the device index you found in the previous step.

4. Wait until the cumulative error value (in PPM) remains more or less the same for three consecutive minutes. Note the last cumulative error value and use it in the `aiscatcher.conf` file.

## Logs

To view the logs of AIS Catcher, use the following commands:

- To view the full log:

  ```bash
  sudo journalctl -u aiscatcher -n 20
  ```

- To view specific parts of the log:

  - To view the received messages:

    ```bash
    sudo journalctl -u aiscatcher -n 200 | grep -o 'received.*'
    ```

  - To view the message rate:

    ```bash
    sudo journalctl -u aiscatcher -n 200 | grep -o 'rate.*'
    ```

  - To view the PPM value:

    ```bash
    sudo journalctl -u aiscatcher -n 30 | awk -F',' '{print $14}'
    ```

  - To view the signal power:

    ```bash
    sudo journalctl -u aiscatcher -n 30 | awk -F',' '{print $13}'
    ```

## Uninstallation

To uninstall AIS Catcher and remove all its files, follow these steps:

1. Stop and disable the AIS Catcher service, and remove the service file:

   ```bash
   sudo systemctl stop aiscatcher
   sudo systemctl disable aiscatcher
   sudo rm /lib/systemd/system/aiscatcher.service
   ```

2. Remove the files and folders related to AIS Catcher:

   ```bash
   sudo rm -rf /usr/share/aiscatcher
   sudo rm /usr/local/bin/AIS-catcher
   sudo userdel aiscat
   ```
