---
layout: default
title: Build Armbian
parent: Operating system
nav_order: 100
---
## Building the Armbian base image

The goal of building the Armbian operating system from source is to create a highly configurable base image, with granular control over kernel features, that can be written on an eMMC or SD card and boots directly into an operational state.

The process to build the Armbian image for the RockPro64 board is mostly automated and follows [Armbian best practices](https://docs.armbian.com/Developer-Guide_Build-Preparation).

It makes use of the Docker containers to perform the build process:

* <https://docs.armbian.com/Developer-Guide_Building-with-Docker/>

The following build instructions have been tested both on Debian-based Linux systems and within Windows PowerShell.

### Requirements

* Regular computer with x86/x64 architecture, 4GB+ memory, 4+ cores
* minimum of 25 GB free space on a fast drive, preferrably SSD
* any operating system that can run Docker containers is supported

### Prepare build environment

To set up and run the build environment in the virtual machine, the following software needs to be installed:

* Git ([download](https://git-scm.com/))
* Docker ([download](https://www.docker.com/get-started)), version >= 18.06.3-ce

All sources and build scripts are contained in this repository, which needs to be cloned locally.
The following commands are executed in the command line, either the Linux terminal or Windows PowerShell.

*Note*: you might need use `sudo` for all `make` commands, depending on your Docker configuration.

* Clone the BitBoxBase repository to a local directory.
  ```bash
  git clone https://github.com/digitalbitbox/bitbox-base.git
  cd bitbox-base
  ```

* Compile custom Go applications
  ```bash
  make docker-build-go
  ```

  (see ["Custom applications / Building Go binaries"](../customapps/go-build.md) for additional information)

### Initial configuration on build

The initial system configuration is set on build and can be altered by setting build options in the file [`armbian/base/build.conf`](https://github.com/digitalbitbox/bitbox-base/blob/master/armbian/base/build.conf).

Available options are described directly in the file and are set to default values.
A few examples of build options you can set:

* `BASE_BITCOIN_NETWORK`: set to `mainnet` or `testnet`
* `BASE_HOSTNAME`: set it to `alice` and your BitBoxBase will be visible as `alice.local` within your network
* `BASE_AUTOSETUP_SSD`: set to `true` to automatically initialize the SSD on first boot
* `BASE_OVERLAYROOT`: set to `true` to make the root filesystem read-only
* ...and many more.

To preserve a local configuration, you can copy the file to `build-local.conf` in the same directory.
This file is excluded from Git source control and overwrites options from `build.conf`.

### Include SSH keys

It is recommended to use SSH keys to access the Base image.
You can include your own keys in the file `armbian/base/config/authorized_keys`, which is excluded from Git versioning.
To include them on build, set `BASE_ADD_SSH_KEYS="true"` in [`armbian/base/build.conf`](https://github.com/digitalbitbox/bitbox-base/blob/master/armbian/base/build.conf).
Please refer to [this article](https://www.atlassian.com/git/tutorials/git-ssh) on how to create your own set of new keys.

### Compile Armbian from source

Now the operating system image can be built. The whole BitBoxBase configuration is contained in [`customize-armbian-rockpro64.sh`](https://github.com/digitalbitbox/bitbox-base/blob/master/armbian/base/customize-armbian-rockpro64.sh) and executed in a `chroot` environment at the end of the build process.

In Linux you can directly run `make`, while in Windows PowerShell you need to run the build script directly with `sh`.
In the following instructions, Windows users just replace `make` with `sh .\build.sh`.

* Start the initial build process.
  ```bash
  cd armbian
  make
  ```

* The resulting image is available in a version subdirectory of `bin/img-armbian/` and can be written to eMMC or SD card using a program like [Etcher](https://www.balena.io/etcher/).
  On the Linux command line you can use `dd`: once the target medium is connected to your computer, get the device name (e.g. `/dev/sdb`).
  Check it carefully, all data on this device will be lost!
  ```bash
  lsblk
  sudo dd if=bin/img-armbian/0.1.0/BitBoxBase-v0.1.0-RockPro64.img of=/dev/sdb bs=64K conv=sync status=progress
  sync
  ```

* After initial build, you can update the image with an adjusted system configuration script, without building Armbian from scratch:
  ```bash
  make update
  ```

* To clean up and remove the build environment, run `make clean`.
