# TLGATE Yocto : build setup, and image creation

## Sources Setup

### Releases and Baselines

Know releases as of Decembre 22 2021 are:

* tlgate-v0.99	:	Released 21/12/2021
* tlgate-v1.00	:	In developement, see [JIRA Release Page](https://jira.open-groupe.com/projects/SEPASSRFNT?selectedItem=com.atlassian.jira.jira-projects-plugin:release-page)
* tlgate-v1.01	:	In developement
* tlgate-v1.10	:	In developement

To build a specific release, you need to set the matching TLGATE_BASELINE.
This is done by selecting the matching config file.

### Tlgate config files (baselines selection)

* the default working baseline is 'develop', this is the "bleeding edge" of the project sources.

```
./oe-layertool-setup.sh -f configs/opengrp-gateway-develop.txt
```

* protected (released) versions will be in the form 'tlgate-vMM.mm'

```
./oe-layertool-setup.sh -f configs/opengrp-gateway-tlgate-v0.99.txt
```

### Developement Images

- `opengrp-gateway-img-dev`: application rootfs and boot folder (kernel+dtb) that addes dev packages and setup to the "release" image, use this for dev and [FT/IT tests](test-strategy.md)

### Specialized Images

- `opengrp-gateway-img-emc`: image for EMC test sessions
- `opengrp-gateway-img-maint`: image for the **maintenance** rootfs and dedicated kernel+dtb. This is used for fallback and production.

### Release Image
- `opengrp-gateway-img-release`: **Release Image**, application rootfs + boot folder (kernel+dtb), use this for [AT tests](test-strategy.md)

## Yocto meta-layers
There are 3 layers specific to the project:
- [meta-tlgate-bsp](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-tlgate-bsp): BSP layer, containing kernel customizations for the hardware
- [meta-tlgate](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-tlgate): motherboard layer containing the DISTRO and images definition, and all mandatory tools
- [meta-sisgateway](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-sisgateway): customer customization (config files, ...)

NOTE: synchronise your repositories regularily, doing a rebase if necessary.
NOTE: the script **sync-tlgate** will try to pull from the origin repository for the project specific meta-layers

## Main Open-groupe packages

The application package is called "tlgate-app"

- `bitbake tlgate-app`: build a given package (only, but including its dependencies)
- `devtool modify tlgate-app`: checkout a package to make changes

It is integrated through a packagegroup file, located in: 

*meta-tlgate/recipes-core/packagegroups/packagegroup-open-gateway.bb*

* tlgate-app has the following recipes layourt in meta-tlgate/recipes-modbus:

```
.
├── tlgate-app
│   ├── tlgate-agn_git.bb
│   ├── tlgate-app
│   │   └── tlgate.service
│   ├── tlgate-app_git.bb
│   ├── tlgate-iec10x.bb
│   └── tlgate-repo.inc
├── tmwscl
│   └── tmwscl_git.bb
└── tmwscl104
    └── tmwscl104_git.bb
```

* tmwscl* ar ethe Triangle Microworks libraries in 2004 (iec101) and 2008 (iec104) versions
* tlgate-agn is a common library, for things ike logs, and parsing the config
* tlgate-iec10* is the public library, for client apps that are launched by tlgate-app. Those client apps implement modbus masters or slaves. 
* sisgateway is recipe name, to build the sisgateway application in repo appgateway
* web-cgi is the recipe to build the cgi-bin applets, and deploy the www contents for the http server (lighttpd)

## The am64xx-tlgate MACHINE

The machine used for the project is "forked" from am64xx-evm. Both MACHINEs can be built, one should:

* build with `MACHINE=am64xx-tlgate` for running on the *tlgate board* a.k.a RUN1 and later RUN2.
* build with `MACHINE=am64xx-evm` for running on the *evaluation board*

Since the device tree for tlgate has no support for sd-card, you won't be able to boot the tlgate software on the evaluation board.

You can switch machines in conf/local.conf, any time: some parts of the build are common, some are specific. In any case, you will get build artifacts either in

`build/arago-tmp-external-arm-glibc/deploy/images/am64xx-tlgate`	for RUNx, see [DFU flashing procedure](../10_production_methods/SEPASSRFNT-61-DFU-flashing.md)

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
Since we are not using ipk we ca just remove this do_create_srcipk task, by commenting in file in *the meta-arago* layer:
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

### Building the CortexR5 SPL and U-boot

Because of the multiple UBOOT_MACHINES, rebuilding the R5 software requires to invoke the proper config:

```
bitbake -c cleansstate u-boot-ti-staging multiconfig:k3r5:u-boot
bitbake  u-boot-ti-staging multiconfig:k3r5:u-boot
bitbake opengrp-gateway-img-dev
```

[Back](toc.md)
