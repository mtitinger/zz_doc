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

[Back](toc.md)
