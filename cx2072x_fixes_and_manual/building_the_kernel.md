# Building the kernel

## Some base info

- This Kernel ist based on the master-branch from https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git (version 4.16 at the time of writing this)
- I Applied the fixes mentioned [here](https://bugzilla.kernel.org/show_bug.cgi?id=115531#c41) to it (See [this PR](https://github.com/heikomat/linux_with_cx2072x/pull/1))
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
- Run `make menuconfig`
- Enable these configurations:
  - ```
    Device Drivers -> Sound card support -> Advanced Linux Sound Architecture -> ALSA for SoC audio support -> CODEC drivers -> Conexant CX2072 CODEC
    ```
  - ```
     Device Drivers -> Sound card support -> Advanced Linux Sound Architecture -> ALSA for SoC audio support -> Intel ASoC SST drivers -> Intel Machine drivers > Baytrail and Cherrytrail with CX2072X codec
     ```

## Building

replace the `8` with the number of cores in your system and run:
```
fakeroot make-kpkg --initrd kernel_image kernel_headers -j 8
```
