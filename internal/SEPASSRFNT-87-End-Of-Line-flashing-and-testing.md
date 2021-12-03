End Of Line Flashing and Testing process ([SEPASSRFNT-87](https://jira.open-groupe.com/browse/SEPASSRFNT-87))
======================================================

This is a draft only, this feature is released in v1.10, this only gives the principles for now.

recommended preliminary reading:
* Bootmodes and JTAG debugging is covered with SEPASSRFNT-63 and [this_doc](SEPASSRFNT-63-JTAG-debug-setup-and-bootmodes.md)
* DFU flashing for SD and eMMC is covered with SEPASSRFNT-61 and [this_doc](SEPASSRFNT-61-DFU-flashing.md)

## General Principle

Upon first poweron, the board is configured by resistors, so that

* the ROM code first attempts to boot from Ethernet0 using BOOTP.
* if BOOTP fails, the ROM code boot the eMMC.

With this config, the Production config is the default, allowing to flash the eMMC, once the eMMC is flashed, the board boots on it.

*Question*: if BOOTP has a long timeout, chose DFU as primary for prod, but this is more difficult to automate.
*Question*: once the software is stable, the eMMC can be pre-flashed, but this creates a delivery process with the contractor, that must be managed. Is it worth the cost, since we will probably update the board on EndOfLine anyways ?

## Production boot and flashing sequence

* step1) premiere sortie de reset : le boot eMMC échoue, la ROM bascule sur le backup (DFU ou encore mieux sur BOOTP)
* step2) le uboot "servi" déroule le flashing de l'eMMC puis le POST (test DDR et eMMC, et surtout UART2 par réponse du PC de prod), active la partition "recue/maintenance"  ("boot-b") et reboot si le POST est OK 
* step3) deuxieme sortie de reset : le boot eMMC sur la partition de maintenance se fait
* step4) le PC host de prod attend le prompt sur la console de debug (UART2) et execute la testsuite du BSP (/opt/ltp) , puis ecrit le rapport de test en EEPROM
* step4.1) si le test est OK : le PC de prod affecte les adresses MAC, PRODUCT TYPE et SERNUM via l'UART et écrit totu ca dans l'EEPROM
* step5) troisième sortie de reset : le boot eMMC boot sur la prochaine partition valide, c'est à dire la partition "application" (a.k.a. "boot-a") 
* step6) la config par défaut de TLGATE lance un rebouclage IEC101 (en cours de mise au point par Thierry) et un slave IEC104, l'opérateur de prod valide sur le PC de prod, que l'appli est fonctionnelle. Ca peut etre aussi automatisé s'il s'agit de la meme config que la CI.
La config finale, est comme toujours spécifique du client final, et faite via l'appli Windows de config (la passerelle sort de prod avec la config de Test Fonctionnel qui est la meme que celle de la CI) 

Il y a quelques briques techno à choisir pour la gestion du valid/invalid des partitions par exemple celle de barebox, comme chez Schneider/IOTB.

## Testing

TODO: writ ethe test strategy document, that should fully define the terminology and process for testing.

### Power On Self Test (POST)

Performed by u-boot in production staged only (specific PROD u-boot, loaded via DFU or BOOTP)

POST goal is to insure that:
* DRR is fine
* UART is fine, and Production PC is responding
* eMMC can be programmed
* optionally : test the EEPROM is present

### BSP Testsuite and VPD/EEPROM data

* Upon booting from the "maintenance"  partition, LTP is run, a repport is generated, and stored to the EEPROM.

IF THE RESULT IS FINE, the following is assigned, and stored to EEPROM;
* SERNUM can be assigned
* MAC address can be assigned
* HOSTNAME can be assigned (for zeroconf for instance, make a hostname based on last MAC@ digits)
* PRODUCT TYPE AND REVISION is assigned
* PRODUCTION DATE (if not in SERNUM)

### Functionnal Loopback test

* The hardware is connected to the same loopback test harness than for CI
* the default Application configuration, is the IEC101-loopback test and a IEC104 slave
* An operator, or automated test scripts from CI are run ansd check that the loopback application works, and IEC104 is polled OK using default IPv4 addresses.


