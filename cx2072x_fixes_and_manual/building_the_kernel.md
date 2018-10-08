# Building the kernel

## Some base info

- This Kernel ist based on the master-branch from https://git.kernel.org/pub/scm/linux/kernel/git/tiwai/sound.git (version 4.16 at the time of writing this)
- I Applied the fixes mentioned [here](https://bugzilla.kernel.org/show_bug.cgi?id=115531#c41) to it (See [this PR](https://github.com/heikomat/linux_with_cx2072x/pull/1))
- These fixes were for older kernels (up to 4.13), so i adjusted them to the best of my knowledge to work in the current kernel
- Keeping this up to date should is possible (updated to 4.19-rc7 at the time of writing this)
- These instructions are written for debian, but they do work on ubuntu and this kernel has also been successfully build for arch (see [here](https://github.com/Grippy98/Asus-E200HA-Linux-Post-Install-Script/issues/30#issuecomment-404034681))

## Building in a Docker container

There is a branch with a dockerfile to build the debian/ubuntu packages for this kernel. I added it so i can build the kernel on macOS, so i don't have to dualboot.

See [this build-script](https://github.com/heikomat/linux/blob/dockerized_deb_build/build.sh), which uses [this Dockerfile](https://github.com/heikomat/linux/blob/dockerized_deb_build/Dockerfile), both from [this branch](https://github.com/heikomat/linux/tree/dockerized_deb_build)

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
