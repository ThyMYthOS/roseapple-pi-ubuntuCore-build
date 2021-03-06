# port of pandaboard board to snappy ubuntu core
build of port for pandaboard to Ubuntu core series 16 [Snappy Ubuntu Core](http://developer.ubuntu.com/snappy/)

## Structure
builder: build board image for Ubuntu core via makefiles, and that includes Gadget snap and Kernel snap.  
prebuild: prebuild iamge for test purpose.

## Requirements
Make sure your build environment is based on `Ubuntu 16.04` or later. Then, you need to install snappy tools, for creating image.

To build all parts, a couple of dependencies are required. On Ubuntu you can install all build dependencies with the following command.

```bash
sudo apt-get update
sudo apt-get install -y build-essential u-boot-tools lzop debootstrap gcc-4.8-arm-linux-gnueabihf device-tree-compiler
sudo apt-get install -y ubuntu-snappy snapcraft
sudo snap install --devmode --edge ubuntu-image
```

Note: snapcraft is broken in Ubuntu 16.04 LTS. See https://bugs.launchpad.net/snapcraft/+bug/1740882

Note: there is no 4.x cross toolchain available in Ubuntu 16.04 LTS. Use the linaro toolchain instead:
```
wget https://releases.linaro.org/components/toolchain/binaries/4.9-2016.02/arm-linux-gnueabihf/gcc-linaro-4.9-2016.02-x86_64_arm-linux-gnueabihf.tar.xz
unxz gcc-linaro-4.9-2016.02-x86_64_arm-linux-gnueabihf.tar.xz
export PATH=$PWD/gcc-linaro-4.9-2016.02-x86_64_arm-linux-gnueabihf/bin/
```

You can test if the right compiler is setup by checking that you get a 4.9 compiler version like:
```
arm-linux-gnueabihf-gcc --version
arm-linux-gnueabihf-gcc (Linaro GCC 4.9-2016.02) 4.9.4 20151028 (prerelease)
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

### Limitation
Ubuntu kernel can't be cross compiled by gcc 5+ arm-linux-gnueabihf, so you have to make soft link to gcc 4.8 related toolchain.

ubuntu-image is still in early days of development, so it has to be installed in devmode and best from edge or beta channel

Generate ssh key-pair if you did not have one

```bash
ssh-keygen -t rsa
```

## Quick Build

Build an image by fetching essential snaps from Ubuntu store
```bash
./build-snappy.sh
```

## Build from scratch
A `Makefile` is provided to build Snappy, Gadget snap, U-Boot, Kernel snap from source. The sources will be cloned into local folders if not there already.

To build it all, just run `make snappy`. This will produce a Snappy image, a gadget snap `pandaboard_x.y_all.snap` and a kernel snap `pandaboard-kernel_x.y.z.snap` for device part, which can be used to build your own Snappy image.

### Custom Image
You can build custom image with extra snaps included, 
```bash
/snap/bin/ubuntu-image \
	-c <channel for core image> \
	--image-size 4G \
	--extra-snaps pandaboard_<gadget snap version>_armhf.snap \
	--extra-snaps pandaboard-kernel_<kernel snap version>_armhf.snap \
	--extra-snaps snapweb \
	--extra-snaps <more addoitional snaps>
	-o pandaboard-<some version to identify new image>.img \
	pandaboard.model
```

### Build U-boot

```bash
make u-boot
```

### Build Gadget snap

```bash
make gadget
```

### Build Kernel snap

```bash
make kernelsnap
```

### Rebuild a snappy
To rebuild the snappy or other parts, just type `make clean` or `make clean-{prefix}`. The prefix will be u-boot, gadget, kernelsnap, etc. 

## Flash to SD card
Before dd, we suggest the SD card storage should be umounted to safely clean up.

```bash
xzcat ${image} | pv | sudo dd of=/dev/${device} bs=32M ; sync
```
## TroubleShooting
Before snapcraft begins to compile the snap, it may pop up an error as below:
```
No valid credentials found. Have you run "snapcraft login"?
```

First, make sure you have Ubuntu One SSO and ability to get 2nd factor from Authenticator, then execute the following command:
```bash
snapcraft login
```
