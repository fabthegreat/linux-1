# Make audio work on cx2072x devices like the Asus E200HA

Most of the following information are from these sources:

- https://bugzilla.kernel.org/show_bug.cgi?id=115531 (a Kernel bugreport)
- https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git (A repository with all the fixes, but for an older kernel)
- https://github.com/Grippentech/Asus-E200HA-Linux-Post-Install-Script (A Repo that contains all these fixes but only for kernels up to 4.12)

You basically need to follow the instructions in [this comment](https://bugzilla.kernel.org/show_bug.cgi?id=115531#c64) from the bugreport. sadly they are not very detailed, so here is what needs to be done:

1. Get the Kernel
   Either download it from this github site, or build it yourself (see "Building the Kernel")
2. Install the kernel by running
   ```
   dpkg -i LINUX_IMAGE_DEB_PACKAGE.deb
   dpkg -i LINUX_HEADERS_DEB_PACKAGE.deb
   ```
3. Have `alsa` and `pulseaudio` installed
4. Copy the `chtcx2072x.conf` and `HiFi.conf` to `/usr/share/alsa/ucm/chtcx2072x`
5. Set `realtime-scheduling = no` in `/etc/pulse/daemon.conf` (see [this issue-comment](https://github.com/Grippentech/Asus-E200HA-Linux-Post-Install-Script/issues/29#issuecomment-355113121))

# Building the kernel

## Some base info

- This Kernel ist a based on the master-branch from https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git (version 4.16-rc5 at the time of writing this), 
- I Applied the fixes mentioned [here](https://bugzilla.kernel.org/show_bug.cgi?id=115531#c41) to it
- These fixes were for older kernels (up to 4.13), so i adjusted them to the best of my knowledge to work in the current kernel
- Keeping this up to date should be possible, though i havent tried it yet
- These instructions are written for debian

## Prerequesites

You need to have the following packages installed:
```
git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc kernel-package
```

## Configuring

- Copy a base config: `cp -v /boot/config-$(uname -r) .config`
- run `make menuconfig`
- select these configurations:
  - Device Drivers -> Sound card support -> Advanced Linux Sound Architecture -> ALSA for SoC audio support -> CODEC drivers -> Conexant CX2072 CODEC
  - Device Drivers -> Sound card support -> Advanced Linux Sound Architecture -> ALSA for SoC audio support -> Intel ASoC SST drivers -> Intel Machine drivers > Baytrail and Cherrytrail with CX2072X codec

## Building

replace the number with your number of cores and run:
```
fakeroot make-kpkg --initrd kernel_image kernel_headers -j 8
```

