# Make audio work on cx2072x devices like the Asus E200HA

## TL;DR

**If You're on Debian or Ubuntu, run this (at your own risk!!)**
```bash
wget -qO- https://gist.github.com/heikomat/3fe272431b44b580c933bfb901a92257/raw | bash
```

## General sound setup:

Most of the following information are from these sources:

- [Kernel Bug #115531](https://bugzilla.kernel.org/show_bug.cgi?id=115531)
- [Repository with the Fixes for an older kernel](https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git)
- [Asus E200HA fix-script-Repo by Grippentech](https://github.com/Grippentech/Asus-E200HA-Linux-Post-Install-Script)

This manual is for debain and derivates of it, though it can ~~probably~~definitely be adapted
for other linux systems (see [here](https://github.com/Grippy98/Asus-E200HA-Linux-Post-Install-Script/issues/30#issuecomment-404034681)). Debian and Ubuntu users can use the [script provided](https://gist.github.com/heikomat/3fe272431b44b580c933bfb901a92257)
above. Its goal is to do the steps described here:

1. Get the Kernel with the cx2072x codec driver and the cx2072x machine driver from [releases](https://github.com/heikomat/linux/releases), or build it yourself (see [building the kernel](building_the_kernel.md#building-the-kernel))

   This tells the Linux how to talk to the hardware.

1. Install the kernel
   ```
   sudo dpkg --install LINUX_IMAGE_DEB_PACKAGE.deb
   sudo dpkg --install LINUX_HEADERS_DEB_PACKAGE.deb
   ```

1. Have `pulseaudio` installed
   ```bash
   sudo apt install pulseaudio
   ```
 
1. Have [`firmware-intel-sound`](https://packages.debian.org/buster/firmware-intel-sound)
   (for debian) or [`linux-firmware`](https://packages.ubuntu.com/de/artful/linux-firmware)
   (for ubuntu) installed.
   
   This contains the firmware for intels sst audio device (`/lib/firmware/intel/fw_sst_22a8.bin`).
   The firmware is the proprietary software that runs within the chip.
   ```bash
   # debian
   sudo apt install firmware-intel-sound

   # ubuntu
   sudo apt install linux-firmware
   ```

1. Copy the configuration files for alsa (`chtcx2072x.conf` and `HiFi.conf` from
   the [chtcx2072x folder](https://github.com/heikomat/linux/tree/cx2072x/cx2072x_fixes_and_manual/chtcx2072x))
   to `/usr/share/alsa/ucm/chtcx2072x` (creating the folder first).

   These tell alsa what driver and codec to use, and how to use them

   ```bash
   sudo mkdir --parents /usr/share/alsa/ucm/chtcx2072x
   cd /usr/share/alsa/ucm/chtcx2072x
   sudo wget "https://raw.githubusercontent.com/heikomat/linux/cx2072x/cx2072x_fixes_and_manual/chtcx2072x/HiFi.conf"
   sudo wget "https://raw.githubusercontent.com/heikomat/linux/cx2072x/cx2072x_fixes_and_manual/chtcx2072x/chtcx2072x.conf"
   ```
1. Set `realtime-scheduling = no` in `/etc/pulse/daemon.conf` (_see [this issue-comment](https://github.com/Grippentech/Asus-E200HA-Linux-Post-Install-Script/issues/29#issuecomment-355113121)_).

   This makes the pulseaudio daemon not die if the audio device is not found instantly

   **via script**
   ```bash
   sudo sed --in-place --regexp-extended --expression='s/;?\s*realtime-scheduling\s*=\s*(yes|no)/realtime-scheduling = no/g' /etc/pulse/daemon.conf
   ```

   **by hand**
   1. Make sure you edit the file as root, for example with `sudo nano /etc/pulse/daemon.conf`
   1. Change `; realtime-scheduling = yes` to `realtime-scheduling = no`
   1. **make sure you removed the `;` at the beginning of the line, this is important!**
1. Reboot

## Possible fixes if audio is still not working:
- Remove possibly existing user-pulse-config with `rm -rf ~/.config/pulse/*`
- Set pulseaudios default device:
  1. Check wich index your non-hdmi audio device has with `pactl list short sinks`
  1. `pacmd set-default-sink 1` (replacing the 1 with the audio-device-index)

## Getting a detailed pulseaudio log, for when debugging is necessary
1. Enable debug-messages for pulseaudio by editing `/etc/pulse/daemon.conf`
   1. Make sure you edit the file as root, for example with `sudo nano /etc/pulse/daemon.conf`
   1. Change `; log-level = notice` to `log-level = debug`
   1. **make sure you removed the `;` at the beginning of the line, this is important!**
1. Reboot
1. Put a log on the Desktop with
   ```bash
   sudo cat /var/log/syslog | grep 'intel\|cx2072x\|pulse\|alsa\|cht\|error' > ~/Desktop/pulselog.txt
   ```
1. Give use the pulselog.txt you now have on your Desktop
