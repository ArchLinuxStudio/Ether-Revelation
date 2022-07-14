# Troubleshooting and solutions

This section describes some of the problems that you are likely to encounter in daily use, and provides solutions.

### The mouse button is insensitive or malfunctioning

Generally speaking, most mice are plug and play, but may experience failure after the 5.14 kernel update. It can be solved by installing the corresponding driver according to your own mouse brand. [[1]](https://openrazer.github.io/#arch)

### It takes a long time to shut down when shutting down

Generally, a message in the form of `A stop job is running for...(1m30s)` will appear on the screen. This is a common problem that the shutdown is stuck for 1 minute and 30 seconds. Generally speaking, this situation is caused by a certain A process is unwilling to stop when it is shut down, and needs to wait until the timeout period is reached to force it to stop. The general solution is to adjust and shorten this waiting time. It is recommended to adjust it from 1 minute and 30 seconds to 30 seconds. 30 seconds is enough for almost all processes to end normally.

Edit `/etc/systemd/system.conf`

```bash
sudo vim /etc/systemd/system.conf
```

Find the `DefaultTimeoutStopSec` item, remove the pound sign in front of it, and assign the value to 30s. Finally execute daemon-reload to make it take effect.

```bash
sudo systemctl daemon-reload
```

The above solution actually only reduces this waiting time, and does not solve the actual problem. If you want to troubleshoot the real cause of the problem, if the message `A stop job is running for...(1m30s)` appears during shutdown, wait patiently for the shutdown to end, then restart the computer and execute the following commands:

```bash
journalctl -p5
```

Press / (slash key) to search for the `Killing` keyword, find the matching line near the time you shut down, you can see which process caused the timeout nearby, and then check what is wrong with this process. Can.

ref: [[1](https://forum.manjaro.org/t/a-stop-job-is-running-for-user-manager-for-uid-1000-during-shutdown/37799)][[ 2](https://unix.stackexchange.com/questions/273876/a-stop-job-is-running-for-session-c2-of-user)]

### How to deal with insufficient disk capacity

Generally use LVM to install Linux system without worrying about this happening. But we are using the traditional ext4 classic partitioning method. In this case, it is generally recommended to set the root directory larger at the beginning of the installation, such as 100G. If the size of the /home partition is not enough, you can install a new hard disk, mount it to the location you want, and then follow the steps of `basic installation` to re-genfstab it.

In addition, if the root directory capacity is insufficient, you can clean the pacman cache from time to time, see [archwiki](https://wiki.archlinux.org/title/Pacman#Cleaning_the_package_cache) for details. If it is too long to read, you can directly use the following command to clean up all cached packages that are not installed, and the synchronization database that is not used.

```bash
sudo pacman -Sc
```

### Software downgrade

Occasionally, on archlinux, the latest version of a certain package has various problems. At this time, the package needs to be downgraded for normal use. The package can be ordinary software or kernel.

```bash
yay -S downgrade
```

Just install this package, and the usage method is also very simple. Just add the package name to be downgraded after downgrade, and then you will be prompted to select the version to be downgraded to, just click.

### An error like unable to lock database occurs when upgrading the system

There may be abnormal shutdown or abnormal program exit when the system is upgraded, or multiple pacman related programs are executed at the same time. Just remove the db lock of pacman

```bash
sudo rm /var/lib/pacman/db.lck
```

### Manual switch mixer

Sometimes the mixer needs to be turned on or off manually for some reason, but currently the mixer cannot be turned off directly in the settings under KDE without shutting down. The following command provides the effect of manually switching the mixer on and off. [[1]](https://unix.stackexchange.com/questions/597736/disabling-kwin-compositor-from-command-line)

```bash
qdbus org.kde.KWin /Compositor suspend #disable

qdbus org.kde.KWin /Compositor resume #Open


```

### Screen overflow: overscan

When connecting some old-fashioned display devices, the phenomenon of [overscan](https://en.wikipedia.org/wiki/Overscan) may appear. Simply put, the TV screen will overflow in four circles, and it will not be displayed. . For Intel HD graphics, you can choose intel panel fitter [[1]](https://askubuntu.com/questions/508358/overscanning-picture-problem-using-hdmi-with-intel-graphics). The last thing is to add to a service to automatically start at boot, and execute [[2]](https://unix.stackexchange.com/questions/397853/how-to-set-a-systemd-unit-to-start-after-loading-the-desktop).

```
sudo intel_panel_fitter -p A -x 1230 -y 700
```
