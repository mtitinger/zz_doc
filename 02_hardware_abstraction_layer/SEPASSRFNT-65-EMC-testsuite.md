# EMC Testsuite

## Feature Status and References

| Technical Notes and Specification | Current [Maturity Grade](../01_developement_methods/SEPASSRFNT-96-developement.md#Maturity Grades)| Comments |
| :---: | :---: | --- |
|[SEPASSRFNT-65](https://jira.open-groupe.com/browse/SEPASSRFNT-65) | MG60 (18/12/21) | development: iperf test integration |


## Feature Description and Domain of Application

This is the tes-suite that is to be used for Electromagnetic Compatibility Tests.

* in order to qualify Tlgate-v6 prototypes (RUN1, RUN2) versus **EMC**
* in order to test Tlgate-v6 prototypes (RUN1, RUN2) versus **robustness (long runs)**

## Requirements

* The testsuite shall test the sub-systems listed below, and also execise them unattended, so electromagnetic emissions can be measured.
* The testsuite shall stimulate the sus-systems at least to the extend of the real application software, under worst case (max load) scenario.

## Tested interfaces

the following interfaces are tested:

| Sub-system | test principle | test script |
| --- | -------- | ------ |
| CPU/PLLs | load with 'stress-ng' hogs | **emm-cpu-ddr** |
| DDR | run 'stress-ng' pattern checking | **emm-cpu-ddr** |
| eMMC | copy (read+write) compare, loop (limited to 100k loops) | **emm-emmc** |
| EEPROM (i2c) | read/write (magic is checked | **emm-i2c-eeprom** |
| GPIO Expander (i2c) | read/write/compare **requires a loopback dongle** | **emm-i2c-gpio** |
| RTC (i2c) | read/write | **emm-i2c-rtc** |
| IHM (leds) | toggle with timer or supported activity metrics | **emm-led-status** |
| QuadUARTs (spi) | echange and compare 128bytes between ttyMAX(2k)<->ttyMAX(2k+1) (ltp) | **emmc-spi-quad-uarts** |
| Ethernet Ports | | |


## Usage

* The testsuite can be installed anyware on the target, preferably in **/home/root/emmc-testsuite**
* It is started with **start-all**
* It is stopped with CTRL+C in start-all, or direct call to **stop-all**

## Dependencies

### Software dependencies

* ltp (with tlgate specific tests)
* stress-ng
* gpio-tools

### Hardware dependencies

* RS485 loopback dongle
* GPIO loopsback dongle

![loopback dongles in place](../images/emc-setup.jpg)

[Back](toc.md)
