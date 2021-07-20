# Building the Yocto project from sources

How to build is documented in the [main Yocto repository](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/opengrp-gateway-sdk).

## Layers
There are 3 layers specific to the project:
- [meta-tlgate-bsp](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-tlgate-bsp): BSP layer, containing kernel customizations for the hardware
- [meta-tlgate](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-tlgate): motherboard layer containing the DISTRO and images definition, and all mandatory tools
- [meta-sisgateway](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/meta-sisgateway): customer customization (config files, ...)

## Images
The available images are the following. Those images are defined in the meta-tlgate layer.
- `opengrp-gateway-img-release`: production image
- `opengrp-gateway-img-dev`: same as opengrp-gateway-img-release with the addition of debug tools, no root password, ...

## The am64xx-tlgate MACHINE

The machine used for the project is "forked" from am64xx-evm.

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

[Back](toc.md)
