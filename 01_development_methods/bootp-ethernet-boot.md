About using BOOTP for Ethernet boot
===================================
## Introduction and Limitations

Full boot using Ethernet only would be very convenient for production stages, because this can be easily automated.
Unfortunately, while the ROM code allows to easily download tiboot3.bin (sysfw and r5 spl), at this time, the R5 SPL has no support for BOOTP.

However, we herein provide some tips to test with BOOTP.

The boot config can be found in [AM64 EVM SK User Guide](https://www.ti.com/lit/ug/spruiy9a/spruiy9a.pdf)

## Host-side setup

A Host PC will act as a server for three procotols:

* DHCP, to assign a IPv4 address
* BOOTP, to reply to the boot request from the ROM, and provide the address of the TFTP server
* TFTP, to reply to the file transfer request.

The more integrated solution for this, is using a fiull featured DCHP server, like _isc-dhcp-server_.
This server can privide all three services by configuration, however there is usually already a potentially conflicting DHCP server on the LAN.

A less integrated yet working and still simple solution, is to have both

* tftpd-hpa configured as regular tftp server
* boopd configured as standalone bootp server

One configured, thtpd-hpa will typically run as: 

```
usr/sbin/in.tftpd --listen --user tftp --address :69 --secure /srv/tftp
```

Bootp shall be launched manually, in standalone mode with:

```
sudo bootpd -s -d -c /srv/tftp -t 20 /srv/tftp/bootptab
```

## Basic Configuration of _bootpd_

An example file would be:

```
$ cat /srv/tftp/bootptab 
.default:\
  hd=./:\
  bf=tiboot3.bin:\
  ip=192.168.1.90:\
  sm=255.255.255.0:\
  sa=192.168.1.13:\
  ha=0xF4844CF95E7A:
```

Where:

* ha is the MAC address set by the ROM (check with Wireshark)
* bf is the file that the ROM expects
* ip is the v4 IP address provided by _bootpd_ using a simplified DHCP service
* sa is the address of the _tftpd-hpa_ server that can provide tiboot3.bin from its file root folder.
* hd is the working directory used with tftpd

## What to do next ?

The ROM will successfully load tiboot3.bin, and start the R5 SPL.
At this point, the R5 SPL has no support for Ethernet boot, and will fail.

It is possible to add Ethernet boot support to the SPL, since other implementations exist, including in the TI family of SoC:

* add the required CONFIG_ item, as per [this link](https://github.com/LeMaker/u-boot/blob/master/doc/SPL/README.am335x-network) (easy)
* write in am65_init.c the "callbacks" to support Ethernet boot, and provide the "VCI strings" that allow patching the requets with a given file: remember SPL has to load both ti-spli.bin and u-boot.img (could be some tricky work: debug, footprint...)
* write a complete bootptab for all three files to upload (easy)


