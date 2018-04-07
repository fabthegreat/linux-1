# Make audio work on cx2072x devices like the Asus E200HA
Most of the following information are from these sources:

- [Kernel Bug #115531](https://bugzilla.kernel.org/show_bug.cgi?id=115531)
- [Repository with the Fixes for an older kernel](https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git)
- [Asus E200HA fix-script-Repo by Grippentech](https://github.com/Grippentech/Asus-E200HA-Linux-Post-Install-Script)

## General sound setup:

1. Get the Kernel with cx2072x support from [here](https://github.com/heikomat/linux/releases), or build it yourself (see [building the kernel](https://github.com/heikomat/linux/blob/cx2072x/cx2072x_fixes_and_manual/building_the_kernel.md#building-the-kernel))
1. Install the kernel by running
   ```
   dpkg -i LINUX_IMAGE_DEB_PACKAGE.deb
   dpkg -i LINUX_HEADERS_DEB_PACKAGE.deb
   ```
1. Have `alsa` and `pulseaudio` installed
1. Copy the `chtcx2072x.conf` and `HiFi.conf` (from the chtcx2072x folder) to `/usr/share/alsa/ucm/chtcx2072x` (creating the folder first)
1. Set `realtime-scheduling = no` in `/etc/pulse/daemon.conf` (see [this issue-comment](https://github.com/Grippentech/Asus-E200HA-Linux-Post-Install-Script/issues/29#issuecomment-355113121))
   1. Make sure you edit the file as root, for example with `sudo nano /etc/pulse/daemon.conf`
   1. Change `; realtime-scheduling = yes` to `realtime-scheduling = no`
   1. **make sure you removed the `;` at the beginning of the line, this is important!**
1. Reboot

## Possible fixes if audio is still not working:
- Remove possibly existing user-pulse-config with `rm -rf ~/.config/pulse/*`
- Set pulseaudios default device:
   1. `pacmd set-card-profile 1 output:HiFi`
   1. `pacmd set-default-sink 1`

## Getting a detailed pulseaudio log, for when debugging is necessary
1. Enable debug-messages for pulseaudio by editing `/etc/pulse/daemon.conf`
   1. Make sure you edit the file as root, for example with `sudo nano /etc/pulse/daemon.conf`
   1. Change `; log-level = notice` to `log-level = debug`
   1. **make sure you removed the `;` at the beginning of the line, this is important!**
1. Reboot
1. Put a log on the Desktop with
   ```
   sudo cat /var/log/syslog | grep 'intel\|cx2072x\|pulse\|alsa\|cht\|error' > ~/Desktop/pulselog.txt
   ```
1. Give use the pulselog.txt you now have on your Desktop

## Detailed manual for Ubuntu
1. Install Ubuntu
1. If you chose Ubuntu 17.10, install pulseaudio 11.1 like this (might be optional, but i did this and it at least didn't break it):
   1. `sudo nano /etc/apt/sources.list`
   1. add this line to the file: `deb http://de.archive.ubuntu.com/ubuntu/ bionic main restricted`
   1. `sudo apt update`
   1. `sudo apt install pulseaudio`
   1. Remove the line you just added from `/etc/apt/sources.list`
   1. `sudo apt update`
   1. reboot
1. Follow all the steps from the [general setup](https://gist.github.com/heikomat/995ab0c5d06e008e6d54411eeb59f75b#general-sound-setup)
1. If sound is not working yet, apply the fixes described [here](https://gist.github.com/heikomat/995ab0c5d06e008e6d54411eeb59f75b#possible-fixes-if-audio-is-still-not-working)

If Audio is still not working, give us a detailed pulseaudio-log as described [here](https://gist.github.com/heikomat/995ab0c5d06e008e6d54411eeb59f75b#getting-a-detailed-pulseaudio-log-for-when-debugging-is-necessary)
