# AOSP+OP-TEE manifest

This repository contains a local manifest that can be used to build an
AOSP build that includes OP-TEE for the hikey board.

## 1. References

* [AOSP Hikey build instructions][1]

## 2. Prerequisites

* Should already be able to build AOSP.  Distro should have necessary
  packages installed, and the repo tool should be installed.  Note
  that AOSP needs to be built with Java.  Also make sure that
  the `mtools` package is installed, which is needed to make the hikey
  boot image.

* In addition, you will need the pre-requisites necessary to build
  optee-os.

  After following the AOSP setup instructions, the following
  additional packages are needed.

```
sudo apt-get install android-tools-adb android-tools-fastboot autoconf \
	automake bc bison build-essential cscope curl device-tree-compiler flex \
	ftp-upload gdisk iasl libattr1-dev libc6:i386 libcap-dev libfdt-dev \
	libftdi-dev libglib2.0-dev libhidapi-dev libncurses5-dev \
	libpixman-1-dev libssl-dev libstdc++6:i386 libtool libz1:i386 make \
	mtools netcat python-crypto python-serial python-wand unzip uuid-dev \
	xdg-utils xterm xz-utils zlib1g-dev python-mako openjdk-8-jdk \
	ncurses-dev realpath android-tools-fsutils dosfstools libxml2-utils
```

## 3. Build steps

```
git clone https://github.com/linaro-swg/optee_android_manifest -b lcr-ref-hikey
cd optee_android_manifest

# for HiKey620
./sync-p.sh
./build-p.sh #or `./build-p.sh -4g` for a 4GB board!

# for HiKey960
./sync-p-hikey960.sh
./build-p-hikey960.sh

# Please make sure there are no errors before flashing!
```

**WARNNING:** `--force-sync` is used which means you might **lose your
work** so save often, save frequent, and save accordingly, especially
before running `sync-p.sh` again!

**EXTREME WARNING:** Do **NOT** use `git clean` with `-x` or `-X` or
`-e` option in `optee_android_manifest`/, else risk **losing all
files** in the directory!!!

**NOTE:** You can add `-squashfs` to `build.sh` option to make
`system.img` size smaller, but this will make `/system` read-only, so
you won't be able to push files to it.

For relatively stable builds, use below instead of `./sync-p.sh`.
```
mkdir logs
./sync.sh -v p -t <hikey|hikey960> -bm <name of a pinned manifest file in archive/> 2>&1 |tee logs/sync-p.log

# e.g.
mkdir logs
./sync.sh -v p -t hikey -bm pinned-manifest-stable_yvr18.xml 2>&1 |tee logs/sync-p.log
```

For other versions, use `./{sync,build}-o.sh` or `./{sync,build}-master.sh`
instead of `./{sync,build}-p.sh`, but these are **NOT MAINTAINED** so
build at your own risk! Note that this applies to **HiKey620 ONLY**!
For HiKey960, only P is currently supported!

## 4. Flashing the image

The instructions for flashing the image can be found in detail under
`device/linaro/hikey{960}/install/README` in the tree.
1. Set jumpers/switches 1-2 and 3-4, and unset 5-6.
2. Reset the board. After that, invoke:

```
# for HiKey620
cp -a out/target/product/hikey/*.img device/linaro/hikey/installer/hikey/
sudo ./device/linaro/hikey/installer/hikey/flash-all.sh /dev/ttyUSBn

# for HiKey960
cp -a out/target/product/hikey960/*.img device/linaro/hikey/installer/hikey960/
sudo ./device/linaro/hikey/installer/hikey960/flash-all.sh /dev/ttyUSBn
```

where the ttyUSBn device is the one that appears after rebooting with
the 3-4 jumper installed.  Note that the device only remains in this
recovery mode for about 90 seconds.  If you take too long to run the
`flash-all.sh` script, it will need to be reset again.

## 5. Partial flashing

The last handful of lines in the `flash-all.sh` script flash various
images.  After modifying and rebuilding Android, it is only necessary
to flash *boot*, *system*, *cache*, and *userdata*.  If you aren't
modifying the kernel, *boot* is not necessary, either.

## 6. Prebuilts (coming soon!)

Available at http://snapshots.linaro.org/android

## 7. Running xtest

Please do NOT try to run `tee-supplicant` as it has already been started
automatically as a service! Once booted to the command prompt, `xtest`
can be run immediately after switching to root, i.e. after running
`su`.

## 8. Enable adb over usb

Boot the device. On serial console:

```
su setprop sys.usb.configfs 1
stop adbd
start adbd
```

## 9. Known issues

- adb currently doesn't work on HiKey960

[1]: https://source.android.com/source/devices.html
