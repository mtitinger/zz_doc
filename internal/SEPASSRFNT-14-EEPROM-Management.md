# EEPROM and Vital Product Data Management

NOTE: For further design details, please also refer to [SEPASSRFNT-14](https://jira.open-groupe.com/browse/SEPASSRFNT-14)

## Introduction

The system features a 2 kiB EEPROM described herein.
Applications, that need to access VPD data stored in the EEPROM (also called NVRAM) will do so by linking and using the tlgate-vpd API described below (WIP).

the API reads an image of the EEPROM (read only) that is created upon system boot by the management application "tlgate-eeprom". During production phase, tlgate-eeprom is also used, to write the initial VPD, for data like serial number, MAC addresses, product type etc...

## tlgate-eeprom Application Usage

### Usage in PRODUCTION 

In production, this app is **run from the maintenance image**, and allows to initialize the EEPROM with the provided data.
iIt will also allow to store the BSP test report.

Contents are written with option:

* **-B binary-file-path**	write the nvram, using a binary image of 2kiB.
* **-J json-file-path**		write the nvram, using a json file, of at most 4kiB.
* **-v**					activate verbose mode.
* **-h**					display help and check layout version (MAGIC)
* **-d**					dump of the contents to local files, in /var/run (this is used on runtime mode mainly).

NOTE: It is very important, that in case of changes, you also change the MAGIC version, to insure the expected scheme, the library, and the app are in sync.

```
{
	"__comment": "This is a dummy JSON template for the EEPROM Scheme 1.O",
	"vpd_magic": "0x4F504E06",
	"vpd_product_type": "0x06241602",
	"vpd_serial_number": "0x20210101",
	"vpd_mac_eth0_msw": "0x00001122",
	"vpd_mac_eth0_lsw": "0x33445560",
	"vpd_mac_eth1_msw": "0x00001122",
	"vpd_mac_eth1_lsw": "0x33445561",
	"vpd_hostname": "sisgateway",
	"vpd_boot_cnt0": 3,
	"vpd_boot_pti0": 0,
	"vpd_boot_cnt1": 3,
	"vpd_boot_pri1": 1,
	"vpd_bsp_test_report": "Test Start Time: Thu Jan 30 11:06:37 2003
	-----------------------------------------
	Testcase                       Result     Exit Value
	--------                       ------     ----------
	mtest01                        PASS       0
	mmstress                       PASS       0
	fork01                         PASS       0
	chdir01A                       PASS       0
	-----------------------------------------------
	Total Tests: 4
	Total Failures: 0
	Kernel Version: 5.10.41
	Machine Architecture: aarch64
	Hostname: lazy"
}
```

### Usage in RUNTIME

In runtime, this app allows to: 

* dump the eeprom contents as a binary file, with **-d**, this is done at startup, and used by the library, for other apps to read the contents

This will create both:

* **/var/run/eeprom.bin**
* **/var/run/eeprom.json**

## Systemd and Yocto Integration

This app is started using a oneshot service, so it can create the read-only image used by the API.
Arguably, this could be done with a udev rule on /sys/class/i2c-dev/i2c-1/device/1-0050/eeprom, but we need more control anyways. The used rule allows us to check for the EEPROM driver readiness, prior to launching the app.

Yocto integation looks like this:

```
├── tlgate-eeprom
│   ├── etc
│   │   └── udev
│   │       └── rules.d
│   │           └── 50-i2c.rules
│   ├── lib
│   │   └── systemd
│   │       ├── system
│   │       │   └── tlgate-eeprom.service
│   │       └── system-preset
│   │           └── 98-tlgate-eeprom.preset
│   └── usr
│       └── bin
│           └── tlgate-eeprom
```

## Building 

Build library first, so it is part of the sysroot.
On a native host PC, builmd typically with:

```
cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/usr .
```

## Testing 
### On PC

first, create a fake image, using the template: 

```
tlgate-eeprom -J res/dummy.json
```

this creates a file /tmp/eeprom.bin, that you can check versus the template: 

```
marc@marc-virtualbox:~/tlgate-eeprom/lib$ od -x /tmp/eeprom.bin 
0000000 4e06 4f50 0000 0000 0101 2021 0000 0000
0000020 0000 0000 0000 0000 0000 0000 0000 0000
*
0004000
```

Then use this file as fake eeprom image, to test dumping:

```
tlgate-eeprom -d
```

Will rewrite this same file, but also parse the binary, and write a json version:

```
marc@marc-virtualbox:~/tlgate-eeprom/lib$ cat /tmp/eeprom.json 
{
	"vpd_magic" : "0x4f504e06",
	"vpd_product_type" : "0x00000000",
	"vpd_serial_number" : "0x20210101",
	"vpd_mac_eth0_msw" : "0x00000000",
	"vpd_mac_eth0_lsw" : "0x00000000",
	"vpd_mac_eth1_msw" : "0x00000000",
	"vpd_mac_eth1_lsw" : "0x00000000",
	"vpd_hostname" : "",
	"vpd_bsp_test_report" : "",
	"vpd_boot_cnt0" : "0x00000000",
	"vpd_boot_pri0" : "0x00000000",
	"vpd_boot_cnt1" : "0x00000000",
	"vpd_boot_pri1" : "0x00000000",
}
```
# Tlgate-vpd Library
## API

We prodivide a trivial API to access the EEPROM in read-only, under runtime.

* *tlgate_vpd_init*, *tlgate_vpd_exit* 	allocate and release API resources.
* *tlgate_vpd_get_serial_number*		get the serial number
* *tlgate_vpd_get_product_type*		get the coded product type
* *tlgate_vpd_get_hostname*		get the default hostname, based on the MAC address
* *tlgate_vpd_get_mac_address*		get the Hardware/MAC address for a given Ethernet Adapter.

## EEPROM Layout 

It is a 24LC16 Microchip I2C 16kbits EEPROM, i.e. 2KiB, byte addresses range from 0x000 to 0x7FF
This is mapped over 8 addresses, from 0x50 to 0x57, i.e. 8 pages of 256 Bytes.

```
root@am64xx-tlgate:~# i2cdetect -y 1 
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                                                 
10:                                                 
20:                                                 
30: -- -- -- -- -- -- -- --                         
40:                                                 
50: UU UU UU UU UU UU UU UU -- -- -- -- -- -- -- -- 
60:                                                 
70:
```

However, the eeprom support in the kernel allows us to access the full 2kiB as a single pseudo file under 

```
/sys/class/i2c-dev/i2c-1/device/1-0050/eeprom
```

Here is the proposed mapping, as a C Structure, as per the library: 

```
typedef struct __attribute__((packed)) _tlgate_eeprom_v6
{
    /*0x000*/ uint32_t vpd_magic;                     /**< Takes value 'O','P','N','6' .>*/
    /*0x004*/ tlgate_product_type_t vpd_product_type; /**< see tlgate_product_type_t below.> */
    /*0x008*/ uint32_t vpd_serial_number;             /**< built as 20YYWWNN, where WW is the week, and YY the year of production, and NN the number of devices in the week.>*/
    /*0x00C*/ uint32_t vpd_mac_eth0_msw;
    /*0x00C*/ uint32_t vpd_mac_eth0_lsw;               /*MAC in LSB*/
    /*0x00C*/ uint32_t vpd_mac_eth1_msw;
    /*0x00C*/ uint32_t vpd_mac_eth1_lsw;              /*MAC in LSB*/
    /*0x01C*/ char vpd_hostname[TLGATE_VPD_HOSTNAME_SZ];
    /*0x05C*/ uint8_t vpd_reserved1[0x100 - 0x1C - TLGATE_VPD_HOSTNAME_SZ];

    /*0x100*/ char vpd_bsp_test_report[TLGATE_VPD_REPORT_SZ];
    /*0x500*/ uint8_t vpd_reserved2[0x7F0 - 0x100 -TLGATE_VPD_REPORT_SZ];

    /*0x7F0*/ uint32_t vpd_boot_cnt0; /*<Will be documented in the Firmware Update Epic.>*/
    /*0x7F4*/ uint32_t vpd_boot_pri0;
    /*0x7F8*/ uint32_t vpd_boot_cnt1;
    /*0x7FC*/ uint32_t vpd_boot_pri1;
} tlgate_eeprom_v6_t ;

typedef union _p
{
    uint32_t id;
    struct
    {
        unsigned char mm; /* 0x6 for V6*/
        unsigned char vv; /* voltage 0x18 for 24v or F9 for 250v*/
        unsigned char cc; /* number of channels, 0x10 */
        unsigned char ee; /* number of ethernet, 0x2 */
    } options;
} tlgate_product_type_t;
```
## Library Integration

This is built with recipe "tlgate-vpd" and will deploy as:

```
├── tlgate-vpd
│   └── usr
│       └── lib
│           ├── libtlgate-vpd.so.1 -> libtlgate-vpd.so.1.0.0
│           └── libtlgate-vpd.so.1.0.0
│
├── tlgate-vpd-dev
│   └── usr
│       ├── include
│       │   └── tlgate-vpd
│       │       └── tlgate-vpd.h
│       ├── lib
│       │   └── libtlgate-vpd.so -> libtlgate-vpd.so.1
│       └── share
│           └── pkgconfig
│               └── tlgate-vpd.pc
```



