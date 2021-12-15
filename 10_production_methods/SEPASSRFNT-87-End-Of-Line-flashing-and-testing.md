End Of Line Flashing and Testing process
======================================================

NOTE: For further design details, please also refer to [SEPASSRFNT-87](https://jira.open-groupe.com/browse/SEPASSRFNT-87)

This is a draft only, this feature is released in v1.10, this only gives the principles for now.

recommended preliminary reading:
* Bootmodes (BM) and JTAG debugging is covered with SEPASSRFNT-63 and [this_doc](SEPASSRFNT-63-JTAG-debug-setup-and-bootmodes.md)
* DFU flashing for SD and eMMC is covered with SEPASSRFNT-61 and [this_doc](SEPASSRFNT-61-DFU-flashing.md)

## General Principle

Upon first poweron, the board is configured by resistors, so that

* the ROM code first attempts to boot eMMC
* if eMMC boot fails, backup on production mode (DFU)

*Question*: once the software is stable, the eMMC can be pre-flashed, but this creates a delivery process with the contractor, that must be managed. Is it worth the cost, since we will probably update the board on EndOfLine anyways ?

## Production boot and flashing sequence

0. The product is assembled, and hooked to the test harness (TDB)

### PHASE 1 : Initial Power On (unflashed hardware)

1. First Power-On-Reset (*POR*) : primary boot mode is eMMC, which will fail, ROM switches over to secondary BM, i.e. DFU
2. Operator runs function "**DFU boot**"
3. "DFU boot" loads tiboot3.bin, tispl.bin **u-boot-maint.img**
4. u-boot-maint.img by default (if not interrupted) run:
	* eMMC partition setup
	* dfu flashing mode
5. Operator runs function "**DFU flash boot0**"
6. "DFU flash boot0" loads again:
	* tiboot3.bin tispl.bin and **u-boot-app.img** are written to boot0 (first hardware boot partition on eMMC)
5. Operator runs function "**DFU flash boot1**"
6. "DFU flash boot1" loads again:
	* tiboot3.bin tispl.bin and **u-boot-app.img** are written to boot1 (second hardware boot partition on eMMC)

### PHASE 2 : Power On Selftest (POST)

10. Operator runs function "**POST**"
11. u-boot leaves dfu listening mode, and runs POST tests :
	* DDR
	* eMMC
	* NVRAM test
	* if it is fine, activates "maintenance" partition (this is done by setting the cnt/prio values in NVRAM)
12. Operator runs function "**DFU flash rootfs**"
13. "DFU flash rootfs" loads WIC file and flashes it to rootfs0 (application rootfs)
14. Operator runs function "**DFU flash maint**"
15. "DFU flash maint" loads and flashes the maintenance image into the eMMC rootfs1 (maintenance rootfs)

### PHASE 3 : Second POR: Product Nomenclature 

20. Second POR : application u-boot selects "maintenance" partition, based on NVRAM counters, and boots it. Note that kernel and device tree are taken from secondary boot is available.
21. Operator runs function "**BSP TestSuite**"
22. "BSP Testsuite" launches (using front-panel UART) the LTP testsuite, if successful, stores it into NVRAM
23. Operator runs function "**Assign VPD**"
24. "Assign VPD" does:
	* assigns the following product values: ID, SERNUM, MAC1, MAC2
	* write them to a JSON file, using the template in SEPASSRFNT-14
	* store to NVRAM using "tlgate-eeprom -J productid-serialnumber.json"
	* maintenance partition is lowered in priority, and application partition is activated (prio/counter in NVRAM)

### PHASE 4 : Final POR: Functionnal testing

30. application u-boot selects "application" rootfs, based on prio/counter in NVRAM
31. default config for TLGATE service launche the IEC101/IEC104 loopback/test scenario
32. Operator checks that test scenarion is fine (modbus poll)

NOTE: The final configuration for the Field depends on the End-User, and is done using the **tlgate.exe** WIndows config application.
This is not done in production, but by Schneider for the End-User, or by the End-User himself.

## Testing

TODO: write the test strategy document, that should fully define the terminology and process for testing.

### Power On Self Test (POST)

Performed by u-boot in production staged only (specific PROD u-boot, loaded via DFU)

POST goal is to insure that:
* DRR is fine
* UART is fine, and Production PC is responding
* eMMC can be programmed
* optionally : test the EEPROM is present

### BSP Testsuite and VPD/EEPROM data

* Upon booting from the "maintenance"  partition, LTP is run, a report is generated, and stored to the EEPROM.

The EEPROM mapping, and management application and API are covered in [EEPROM Management](SEPASSRFNT-14-EEPROM-Management.md)

### Functionnal Loopback test

* The hardware is connected to the same loopback test harness than for CI
* the default Application configuration, is the IEC101-loopback test and a IEC104 slave
* An operator, or automated test scripts from CI are run and check that the loopback application works, and IEC104 is polled OK using default IPv4 addresses.


