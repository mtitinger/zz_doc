# Building the Yocto project from sources

Main "top" directory is based on Yocto Dunfell for TI AM64x SoC. It is a fork from [git://arago-project.org/git/projects/oe-layersetup.git](git://arago-project.org/git/projects/oe-layersetup.git).

## Pre-requisites
(copy/pasted from https://software-dl.ti.com/processor-sdk-linux/esd/AM64X/07_03_00_02/exports/docs/linux/Overview_Building_the_SDK.html)

```
$ sudo apt-get install build-essential autoconf automake bison flex libssl-dev bc u-boot-tools python diffstat texinfo gawk chrpath dos2unix wget unzip socat doxygen libc6:i386 libncurses5:i386 libstdc++6:i386 libz1:i386 g++-multilib python3-distutils
$ sudo dpkg-reconfigure dash

$ wget https://developer.arm.com/-/media/Files/downloads/gnu-a/9.2-2019.12/binrel/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz
$ sudo tar -Jxvf gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz -C /opt
$ wget https://developer.arm.com/-/media/Files/downloads/gnu-a/9.2-2019.12/binrel/gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu.tar.xz
$ sudo tar -Jxvf gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu.tar.xz -C /opt
```

## Bootstraping for OpenGroupe

* A build server is setup here: 10.20.3.10 (within the Opengroupe VPN)
* Every developer has to create his own build directory in `/opt/build/{trigram}`

For your comfort, the following git parameters can be set:
* preferably use vim over nano : sudo update-alternatives --config editor

### git aliases and user config (for sign-off)

Adapt the following to your case, in ~/.gitconfig

```
[alias]
        st = status
        ci = commit
        br = branch
        co = checkout
[user]
        name = Marc Titinger
        email = marc.titinger@open-groupe.com
```

### Dealing with git protocol restrictions (WIP)

You can use the following substitutions in ~/.gitconfig

```
[url "https://git.yoctoproject.org/git"]
        insteadOf = git://git.yoctoproject.org

[url "http://git.ti.com/git"]
        insteadOf = git://git.ti.com

[url "http://"]
        insteadOf = git://
```

## Install sources
```
./oe-layertool-setup.sh -f configs/opengrp-gateway-sdk.txt
cd build
. conf/setenv
bitbake opengrp-gateway-img-release
```

**Note:**
For installing a given version:
```
 ./oe-layertool-setup.sh -f configs/opengrp-gateway-vMM.nn.bb.txt
```

## Layers
There are 3 layers specific to the project:
- [meta-tlgate-bsp](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-tlgate-bsp): BSP layer, containing kernel customizations for the hardware
- [meta-tlgate](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-tlgate): motherboard layer containing the DISTRO and images definition, and all mandatory tools
- [meta-sisgateway](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-sisgateway): customer customization (config files, ...)

## Images and packages
The available images are the following. Those images are defined in the meta-tlgate layer.
- `bitbake opengrp-gateway-img-release`: production image
- `bitbake opengrp-gateway-img-dev`: same as opengrp-gateway-img-release with the addition of debug tools, no root password, ...
- `bitbake tlgate`: build a given package (only, but including its dependencies)
- `devtool modify tlgate`: checkout a package to make changes

## The am64xx-tlgate MACHINE

The machine used for the project is "forked" from am64xx-evm. Both MACHINEs can be built, one should:

* build with `MACHINE=am64xx-evm` for running on the *evaluation board*
* build with `MACHINE=am64xx-tlgate` for running on the *tlgate board* a.k.a RUN1 and later RUN2.

Since the device tree for tlgate has no support for sd-card, you won't be able to boot the tlgate software on the evaluation board.

You can switch machines in conf/local.conf, any time: some parts of the build are common, some are specific. In any case, you will get build artifacts either in

* `build/arago-tmp-external-arm-glibc/deploy/images/am64xx-evm`		for devkit flashing procedure, is dd'ing to and external SDCard.
* `build/arago-tmp-external-arm-glibc/deploy/images/am64xx-tlgate`	for RUNx (flashing procedure To Be Documented)

### The need for overrides for u-boot and ti-sci-fw
We have .bbappend files for those recipes, which are just copy-pasted from the TI recipes in order to provide the variables and functions with the proper machine override (i.e. the _am64xx-tlgate suffix).
If we don't provide those, the variables are not defined at all.

For example:
```json
DEPENDS_append_am64xx-tlgate-k3r5 = " virtual/bootloader"
...
do_install_am64xx-tlgate-k3r5() {
	...
}
```

### The use of multiconfig
The Yocto project makes use of multiconfig to handle the fact that we're building some recipes for 2 cores: A5 (running Linux) and R5. We build u-boot for the 2 different cores and configurations.
Even if we don't deploy an application to the R5 core, we **do** need to build its bootloader firmware because R5 boots first and in turn loads the bootloader for A5.
This is detailed in u-boot sources, in the `./board/ti/am65x/README` (there's no readme for am64x).

That's the reason why we define the am64xx-tlgate MACHINE in the meta-tlgate-bsp layer, but also the am64xx-tlgate-k3r5 secondary machine.

## Tips and Tricks

### About using devtool

I am finding that "devtool modify" can be broken because of the create_scripk errors.
Since we are not using ipk we ca just remove this do_create_srcipk task, by commenting in file:

```patch
diff --git a/meta-arago-distro/recipes-kernel/linux/copy-defconfig.inc b/meta-arago-distro/recipes-kernel/linux/copy-defconfig.inc
index 10ecf8ea..73b14430 100644
--- a/meta-arago-distro/recipes-kernel/linux/copy-defconfig.inc
+++ b/meta-arago-distro/recipes-kernel/linux/copy-defconfig.inc
@@ -19,4 +19,4 @@ do_configure_append() {

 # Move create_srcipk task so that the release defconfig is included.
 deltask do_create_srcipk
-addtask create_srcipk after do_configure before do_compile
+#addtask create_srcipk after do_configure before do_compile
```

### Create a bootable SD-card
First identify your block device using lsblk for instance, to prevent to alter the wrong filesystem.

```json
cd build/arago-tmp-external-arm-glibc/deploy/images/am64xx-tlgate
sudo umount /dev/mmcblk0p?
sudo dd bs=4M if=opengrp-gateway-img-release-am64xx-tlgate.wic of=/dev/mmcblk0 status=progress && sync
OR:
sudo bmaptool copy opengrp-gateway-img-release-am64xx-tlgate.wic /dev/mmcblk0 && sync
```

### Building a custom Linux kernel
There are different ways to do this, here is at least one solution:
1) clone the Kernel repository (https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/ti-linux-kernel, use the tlgate-5.4 branch)
2) make your changes, and **commit them**
3) In Yocto sources tree, patch the kernel recipe (`sources/meta-tlgate-bsp/recipes-kernel/linux/linux-ti-staging_%.bbappend`) this way:
```
@@ -7,6 +7,6 @@
 SRCREV = "${AUTOREV}"
 
 # Note: defconfig is here just to point kernel config to the in-repo config file
 SRC_URI = " \
-       git://git@git.boost.open.global:443/schneider-electric/passerelle_refonte/Software/bsp/ti-linux-kernel.git;protocol=ssh;branch=${BRANCH} \
+       git://<path to your kernel clone: absolute path including leading />;protocol=file;branch=${BRANCH} \
        file://defconfig \
```
4) Rebuild Kernel: the command to rebuild the whole recipe (including device tree) is `bitbake -c cleanall linux-ti-staging && bitbake linux-ti-staging`.
5) You can get resulting files in `build/arago-tmp-external-arm-glibc/work/am64xx_tlgate-linux/linux-ti-staging/5.4.106+gitAUTOINC+XXXXX-r0a.arago5_psdkla_8/image`.

**Notes:**
- The kernel config is `arch/arm64/configs/tisdk_am64xx-tlgate_defconfig`
- The device tree is `arch/arm64/boot/dts/ti/k3-am642-tlgate.dts`

### using SSHFS

In your client VM, you need to install sshfs

```
sudo apt install sshfs
```
in /etc/fstab, add an entry, I recommend you use the same path than on the build server
so that any path sensitive script you make will also work.

```
mti20447@10.20.3.10:/opt/build/mti20447  /opt/build/mti20447  fuse.sshfs  port=22,user,noatime,reconnect,_netdev,allow_other  0  0
```

Make sure to generate an rsa key (w/o password) and copy your key to your user on the build server:

```
ssh-keygen ...
ssh-copy-id -i ~/.ssh/id_rsa.pub yourtrigram@10.20.3.10
```

[Back](toc.md)
