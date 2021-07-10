# Useful commands

## Check OS version in Linux

```bash
ubuntu@pik8s-controller:~$ cat /etc/os-release
NAME="Ubuntu"
VERSION="20.04.2 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.2 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

```bash
ubuntu@pik8s-controller:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description: Ubuntu 20.04.2 LTS
Release: 20.04
Codename: focal
```

```bash
:~$ hostnamectl
```
## PoE HAT Fan Speed adjustment

By default, the fans on the PoE HATs spin up around 40 C, and in my case seemed to be running continuously. 
For Ubuntu 20.04, follow the steps as answered in [this post](https://raspberrypi.stackexchange.com/a/112094) 

The fan config is in `/sys/class/thermal/cooling_device0/`, if you `cat /sys/class/thermal/cooling_device0/type`, it should be "rpi-poe-fan".

Once you've confirmed that, use this udev rule as an example:

```bash
SUBSYSTEM=="thermal"
KERNEL=="thermal_zone0"

# If the temp hits 75c, turn on the fan. Turn it off again when it goes back down to 70.
ATTR{trip_point_3_temp}="75000"
ATTR{trip_point_3_hyst}="5000"
#
# If the temp hits 78c, higher RPM.
ATTR{trip_point_2_temp}="78000"
ATTR{trip_point_2_hyst}="2000"
#
# If the temp hits 80c, higher RPM.
ATTR{trip_point_1_temp}="80000"
ATTR{trip_point_1_hyst}="2000"
#
# If the temp hits 81c, highest RPM.
ATTR{trip_point_0_temp}="81000"
ATTR{trip_point_0_hyst}="5000"
```

Place it in `/etc/udev/rules.d/50-rpi-fan.rules`. To apply udev rules, issue

```bash
sudo udevadm control --reload-rules && udevadm trigger
```

### To check the current state of the fan 

```bash
cat /sys/class/thermal/cooling_device0/cur_state
```

```
0 ==> Fan is not running
1 ==> Fan is running
```

### To turn on/off the fan manually

```bash
echo '0' | sudo tee /sys/class/thermal/cooling_device0/cur_state
```

### References
- https://raspberrypi.stackexchange.com/questions/98078/poe-hat-fan-activation-on-os-other-than-raspbian
- <https://www.jeffgeerling.com/blog/2021/taking-control-pi-poe-hats-overly-aggressive-fan>
- <https://www.raspberrypi.org/forums/viewtopic.php?t=276805>
