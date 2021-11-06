# TLGATE Yocto : Onetime Setup Instruction and Introduction

Main "top" directory is based on Yocto Dunfell for TI AM64x SoC. It is a fork from [git://arago-project.org/git/projects/oe-layersetup.git](git://arago-project.org/git/projects/oe-layersetup.git).

* We have setup a build server (10.20.3.10), but you can also build yocto locally on your machine, in this case, you need to do the following one time.
* the "for YOU"sections, you need to do even if your are building using the Open-groupe project server.


## Pre-requisites (for the SERVER)
(copy/pasted from https://software-dl.ti.com/processor-sdk-linux/esd/AM64X/07_03_00_02/exports/docs/linux/Overview_Building_the_SDK.html)

```
sudo apt-get install build-essential autoconf automake bison flex libssl-dev bc u-boot-tools python
 diffstat texinfo gawk chrpath dos2unix wget unzip socat doxygen libc6:i386 libncurses5:i386 libstdc++6:i386
 libz1:i386 g++-multilib python3-distutils chrpath diffstat gawk git gobject-introspection graphviz
```

Bash must be used by defautl, not dash: run this and select "no"

```
sudo dpkg-reconfigure dash
```

The TI toolchains are not built, instead they are taken from Linaro: 

```
wget https://developer.arm.com/-/media/Files/downloads/gnu-a/9.2-2019.12/binrel/gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz
sudo tar -Jxvf gcc-arm-9.2-2019.12-x86_64-arm-none-linux-gnueabihf.tar.xz -C /opt
wget https://developer.arm.com/-/media/Files/downloads/gnu-a/9.2-2019.12/binrel/gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu.tar.xz
sudo tar -Jxvf gcc-arm-9.2-2019.12-x86_64-aarch64-none-linux-gnu.tar.xz -C /opt
```

## Bootstraping for OpenGroupe (for the SERVER)

* A build server is setup here: 10.20.3.10 (within the Opengroupe VPN)
* Every developer has to create his own build directory in `/opt/build/{trigram}`

For your comfort, the following git parameters can be set:
* preferably use vim over nano : sudo update-alternatives --config editor

### git aliases and user config (for YOU)

Adapt the following to your case, in ~/.gitconfig
It will create some aliases, and setup substitutions to circumvent ports restrictions (git uses 9418 tcp)

```
[alias]
        st = status
        ci = commit
        br = branch
        co = checkout
[user]
        name = Marc Titinger
        email = marc.titinger@open-groupe.com

[url "https://git.yoctoproject.org/git"]
        insteadOf = git://git.yoctoproject.org

[url "http://git.ti.com/git"]
        insteadOf = git://git.ti.com

[url "http://"]
        insteadOf = git://
```
### using SSHFS (for YOU)

In your client VM, you need to install sshfs

```
sudo apt install sshfs
```
in /etc/fstab, add an entry, I recommend you use the same path than on the build server
so that any path sensitive script you make will also work.

*Of course change the values, tu adapt to your user*

```
mti20447@10.20.3.10:/opt/build/mti20447  /opt/build/mti20447  fuse.sshfs  port=22,user,noatime,reconnect,_netdev,allow_other  0  0
```

Make sure to generate an rsa key (w/o password) and copy your key to your user on the build server:

```
ssh-keygen ...
ssh-copy-id -i ~/.ssh/id_rsa.pub yourtrigram@10.20.3.10
```

[Back](toc.md)
