# TLGATE Yocto : building the SDK

## Introduction
The SDK is useful mainly for debugging, and gettings backtraces in gdb, but should not be used mainly for
coding and compiling, use devtool modify instead.

## Creation of the SDK installer

It can be build based on the dev image, with:

```
bitbake opengrp-gateway-img-dev -c populate_sdk
```

However _libatomic-dev_ must be removed froom the SDK packagegroup, by adding this to the conf/local.conf

```
# In order to built the SDK
RDEPENDS_packagegroup-core-standalone-sdk-target_remove = "libatomic-dev" 
```

## SDK installation and usage

* The SDK installer is generated as *build/arago-tmp-external-arm-glibc/deploy/sdk/tlgate-sdk-5.10-zeus-toolchain-5.10-zeus.sh*
* it default installation location is */usr/local/tlgate-sdk-x86_64*, but I recommend installing it under */opt/tlgate-sdk-x86_64*
* once installed you can source the dev environment with:

```
source /opt/tlgate-sdk-x86_64/environment-setup-aarch64-linux
```

[Back](toc.md)
