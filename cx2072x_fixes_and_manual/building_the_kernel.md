# Building the kernel

## Some base info

- This Kernel ist based on the current `topic/soc-cx2072x`-branch from [here](https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git?h=topic%2Fsoc-cx2072x-5.1) (`topic/soc-cx2072x-5.1` at the time of writing this)
- I added (and updated) alsa ucm files mentioned in [this](https://bugzilla.kernel.org/show_bug.cgi?id=115531#c41) bugreport (Thank you  [@roethigj](https://github.com/roethigj) for [informing me](https://github.com/heikomat/linux/issues/8#issuecomment-493793106) about takashis updated patches and the required adjustments to the ucm files!)
- These instructions are written for debian, but they do work on ubuntu and this kernel has also been successfully build for arch (see [here](https://github.com/Grippy98/Asus-E200HA-Linux-Post-Install-Script/issues/30#issuecomment-404034681))

## Building in a Docker container

There is a branch with a dockerfile to build the debian/ubuntu packages for this kernel. I added it so i can build the kernel on macOS, so i don't have to dualboot.

See [this build-script](https://github.com/heikomat/linux_cx2072x_build/blob/master/build_debian.sh), which uses [this Dockerfile](https://github.com/heikomat/linux_cx2072x_build/blob/master/Dockerfile_debian), both from [this repo](https://github.com/heikomat/linux_cx2072x_build)

Be prepared for the docker image to get about 27GB in size. As you can see in the Dockerfile, it builds in an ubuntu container by pulling from the cx2072x-branch. you might want to adjust the number of threads used.

## Building the conventional way

### Prerequesites

You need to have the following packages installed:
```
git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc kernel-package
```

### Configuring

- Copy a base config: `cp -v /boot/config-$(uname -r) .config`
- Run `make menuconfig`
- Enable these configurations:
  - ```
    Device Drivers -> Sound card support -> Advanced Linux Sound Architecture -> ALSA for SoC audio support -> CODEC drivers -> Conexant CX2072 CODEC
    ```
  - ```
     Device Drivers -> Sound card support -> Advanced Linux Sound Architecture -> ALSA for SoC audio support -> Intel ASoC SST drivers -> Intel Machine drivers > Baytrail and Cherrytrail with CX2072X codec
     ```
- Remove the string in this configuration:
  - ```
     Cryptographic API -> Certificates for signature checking -> Provide system-wide ring of trusted keys -> Additional X.509 keys for default system keyring
    ```

### Building

replace the `8` with the number of cores in your system and run:
```
make deb-pkg -j 8
```
