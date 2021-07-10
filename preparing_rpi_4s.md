# Preparing Raspberry Pi 4

This section will cover the details about preparing the raspberry pi 4 nodes before we can setup a k8s cluster.

## Setting up the Raspberry Pis

## Assemble the cluster case

## Updating the Bootloader

Using Raspberry Pi Imager to update the bootloader (recommended)

Raspberry Pi Imager provides a GUI for updating the bootloader and selecting the boot mode.

1. Download [Raspberry Pi Imager](https://www.raspberrypi.org/downloads/)
1. Select a spare SD card. The contents will get overwritten!
1. Launch Raspberry Pi Imager
Select Misc utility images under Operating System
1. Select Bootloader
1. Select a boot-mode i.e. SD (recommended), USB or Network.
1. Select SD card and then Write
1. Boot the Raspberry Pi with the new image and wait for at least 10 seconds.
1. The green activity LED will blink with a steady pattern and the HDMI display will be green on success.
1. Power off the Raspberry Pi and remove the SD card.

Reference: <https://www.raspberrypi.org/documentation/hardware/raspberrypi/booteeprom.md#imager>

## Verify the Bootloader is up-to-date

```bash
sudo rpi-eeprom-update
```

The output should look something like this:

```bash
BCM2711 detected
VL805 firmware in bootloader EEPROM
BOOTLOADER: up-to-date
CURRENT: Thu Apr 29 16:11:25 UTC 2021 (1619712685)
 LATEST: Thu Sep  3 12:11:43 UTC 2020 (1599135103)
 FW DIR: /lib/firmware/raspberrypi/bootloader/default
VL805: up-to-date
CURRENT: 000138a1
 LATEST: 000138a1
```

## Install Ubuntu Server on USB Flash Drive

1. Download the Ubuntu Image for RPi 4 from the Ubuntu official website
   - [Ubuntu Server 20.04.2 LTS](https://cdimage.ubuntu.com/releases/20.04.2/release/ubuntu-20.04.2-preinstalled-server-arm64+raspi.img.xz)
1. Flash the image onto the UBB drive using Raspberry Pi Imager.
1. Remove and re-insert the USB flash drive again in your PC/Mac.
1. Open a Terminal and cd to `/Volumes/system-boot`
1. Run following to decompress the vmlinuz on the boot partition

   ```bash
   gzcat vmlinuz > vmlinux
   ```

1. Update the file `config.txt` as follows for the [pi4] section

   ```bash
   [pi4]
   max_framebuffers=2
   dtoverlay=vc4-fkms-v3d
   boot_delay
   kernel=vmlinux
   initramfs initrd.img followkernel
   ```

1. Add a new script to the boot partition called auto_decompress_kernel with the following: (NOTE - Reference [here](https://www.raspberrypi.org/forums/viewtopic.php?t=281152))

    ```bash
    #!/bin/bash -e

    #Set Variables
    BTPATH=/boot/firmware
    CKPATH=$BTPATH/vmlinuz
    DKPATH=$BTPATH/vmlinux

    #Check if compression needs to be done.
    if [ -e $BTPATH/check.md5 ]; then
        if md5sum --status --ignore-missing -c $BTPATH/check.md5; then
        echo -e "\e[32mFiles have not changed, Decompression not needed\e[0m"
        exit 0
        else echo -e "\e[31mHash failed, kernel will be compressed\e[0m"
        fi
    fi

    #Backup the old decompressed kernel
    mv $DKPATH $DKPATH.bak

    if [ ! $? == 0 ]; then
        echo -e "\e[31mDECOMPRESSED KERNEL BACKUP FAILED!\e[0m"
        exit 1
    else  echo -e "\e[32mDecompressed kernel backup was successful\e[0m"
    fi

    #Decompress the new kernel
    echo "Decompressing kernel: "$CKPATH".............."

    zcat $CKPATH > $DKPATH

    if [ ! $? == 0 ]; then
        echo -e "\e[31mKERNEL FAILED TO DECOMPRESS!\e[0m"
        exit 1
    else
        echo -e "\e[32mKernel Decompressed Succesfully\e[0m"
    fi

    #Hash the new kernel for checking
    md5sum $CKPATH $DKPATH > $BTPATH/check.md5

    if [ ! $? == 0 ]; then
        echo -e "\e[31mMD5 GENERATION FAILED!\e[0m"
        else echo -e "\e[32mMD5 generated Succesfully\e[0m"
    fi

    #Exit
    exit 0
    ```

1. Start the Raspberry Pi without the SD card and the USB driver inserted. Make sure to plug-in an ethernet cable in the RJ45 port. After the server is booted first, the default login credentials are as follows:

    ```bash
    ubuntu / ubuntu
    ```

    You will be prompted to change the default password to something more secure.

1. **NOTE** Once your ubuntu is booted, there could be an automated daemon thread `unattended-upgr` that may have kicked-off the apt upgrade for critical security upgrades. Before you reboot the pi, make sure to complete the following steps otherwise the updated kernel will remain uncompressed causing your pi to fail to boot correctly. A workaround would be to repeat the Step 5. However the following is a more recommended appproach (until this is fixed in the next LTS release)

1. Create a script in the `/etc/apt/apt.conf.d` directory and name it as `999_auto_decompress_rpi_kernel`. Add the following to the script

    ```bash
    echo 'DPkg::Post-Invoke {"/bin/bash /boot/firmware/auto_decompress_kernel"; };' | sudo tee /etc/apt/apt.conf.d/999_auto_decompress_rpi_kernel
    ```

    ```bash
    DPkg::Post-Invoke {"/bin/bash /boot/firmware/auto_decompress_kernel"; };
    ```

    This will setup a trigger to decompress the kernel whenever the kernel is updated in future.

1. Change the hostname to identify the nodes

    ```bash
    sudo nano /etc/hostname
    ```

    Set the hostname for the controller node to `pik8s-controller`.

    Set the hostname for the worker nodes to `pik8s-node-01`, `pik8s-node-02` ...

1. Also add an entry to for ip address `127.0.1.1` to the `/etc/hosts` file for the hostnames added above e.g.

    ```bash
    127.0.0.1 localhost
    127.0.1.1 pik8s-controller

    # The following lines are desirable for IPv6 capable hosts
    ::1 ip6-localhost ip6-loopback
    fe00::0 ip6-localnet
    ff00::0 ip6-mcastprefix
    ff02::1 ip6-allnodes
    ff02::2 ip6-allrouters
    ff02::3 ip6-allhosts
    ```

1. Create a new user on the pis and add the newly created user to the `sudo` group.

    ```bash
    sudo adduser pik8su
    ```

    ```bash
    sudo usermod -aG sudo pik8su
    ```

1. Configure boot options

    ```bash
    sudo nano /boot/firmware/cmdline.txt
    ```

    Append following at the end of the line

    ```bash
    cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1
    ```

1. Reboot your PIs

    ```bash
    sudo reboot
    ```
