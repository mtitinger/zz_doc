# Cyber Security Requirements

NOTE: For further design details, please also refer to [SEPASSRFNT-46](https://jira.open-groupe.com/browse/SEPASSRFNT-46)

**This product will undergo Schneider Electric cysec acceptance review, and must be ready to comply to IEC62443 Securiy Level 1 (SL1)**

## Cyser Security Management Plan

### CySec Governance Overview

Cybersecurity threat mitigation for embedded systems is best achieved by following a formal CySec Development Lifecycle, meaning that this should not be a late story or corrective action in the development cycle, but involves specification abd review at all steps of the development. In few words: a Formal Cyber Security Management Plan (**CSMP**) must be written, and complied to during the complete product lifecycle.

In the context of this project, because we are upgrading a former existing project with absolutely no CySec mitigation, we implement CySec in a "relaxed" way, following two principles:

1. Implement all common-sense features that are standard in any state-of-the-art low-security embedded product:
	* Users and Groups
	* firewall rules
	* dbus (IPCs) interfaces allow/deny rules
	* removal of all ports, protocol, interfaces that are not used

2. Insure that the Hardware and the Software do not prevent to achieve IEC-62443/SL1 compliance, if/when required by the customer, while we don't certify the product for this standard for now.
	* list and understand the practical requirements of the Standard
	* when possible implement them
	* if a substantial amount of development time is required for a given requirement, don't implement it, but clearly state how this could be done (not blocking points)

### CySec Reviews and Referents 

We do not plan specific reviews, and do not appoint any project referent, other than those already part of Open-Groups internal CySec Management Plan (RED?)

### Threat Model for SISGAteway

Based on Microsoft Model

#### Identification (STRIDE)

we try to map threats as follows

| Threat Type | General Principle | SISGateway Applicable interfaces |
| --- | --- | --- |
| **S**poofing |  authentication : pretend to be s/o else | Front Panel UART logging, Ethernet protocols, Modbus RS, IEC101 |
| **T**empering | integrity : change, make run your code | all of the above, plus internal interfaces : USB, JTAG. Hacking of eMMC, NVRAM |
| **R**epudiation | failure to adopt controls to properly track and log users' actions | all of the above |
| **I**nformation Disclosure | failure to protect private data like keys, peoples, or strategy data (business or security related) | website, documentation, filesystem |
| **D**enial of Service | failure to protect against exploits and spamming of interfaces and protocols, leading to lack of availability of some service | Ethernet protocols, UART-console |
| **E**levation of privilege | failure to prevent to gain Privileged Access through Credential exploitation, Vulnerabilities, Misconfiguration, Malware, Social engineering | all of the above |

#### Ranking (DREAD)

* For simplicity, we assume users and groups are enforced, as in section "Users and Groups", we also assume that unused network protocols are filtered out using netfilter/iptables.
* We do not address Attacks that involve tempering with the hardware (unsolder parts, connect a JTAG probes etc...)

| Attack | **D**amage potential | **R**eproducibility | **E**xploitation (skill needed) | **A**ffected user | **D**iscoverability |
| --- | --- | --- | --- | --- | --- |
| logging as "root" in rootfs | this device: rootfs no longer bootable, total loss of functionality | easy | easy (social engineering) | OPERATOR (End-user), CONFIGURATOR | immediate |
| logging as "root" in maintenance | this device: total loss of functionality, device identity, and on-site recovery  | easy | easy (social engineering) | OPERATOR (End-user), CONFIGURATOR | immediate |
| HTTP attack, random program execution via CGI | loss of web interface, misconfiguration | medium | medium | OPERATOR, CONFIGURATOR | variable |
| HTTP DoS requests spamming | loss of web interface reactivity, misconfiguration | easy | easy | OPERATOR, CONFIGURATOR, L3SUPPORT | variable |
| IEC104 exploits | TBD | hard | high (specific) | OPERATOR (TDB) | TBD |
| IEC101 exploits | TBD | hard | high (specific) | OPERATOR (TDB) | TBD |
| Modbus RS exploits | TBD | hard | high (specific) | OPERATOR (TDB) | TBD |
| Intruding in u-boot | this device: total loss of functionality, device identity, and on-site recovery | medium | medium | All Users | variable |
| USB mass storage mounting with harmful software | this device: rootfs no longer bootable, total loss of functionality | medium | high (combine with other attacks to get privilege | OPERATOR, CONFIGURATOR | variable |
| Tempering with bootmodes | this device: total loss of functionality, device identity, and on-site recovery | medium | medium (specific knowledge of the platform h/w and s/w) | All users | variable |
| Tempering with firmware-update bundles | this device: total loss of functionality | medium | medium (specific knowledge of the platform h/w and s/w) | All users | variable |

### Applicable Normative Requirements

We only address Threads Ranked as **easy to reproduce, and requiring low skills** (human error), as per IEC-62443/SL1.

### SL1 Definition

SL1 matches involuntary attacks, performed by accident by an person not skilled in system hacking.

![Cysec Security Levels](../images/cysec-security-levels.png) 

### SL1 Mitigations

The required mitigations are listed here:

![Cysec Security Levels](../images/cysec-sl1-requirements.png)

## SISGateway Mitigation Specification

### Users and Groups management

Users to Create: 

| User Name |  Type | Description |
| --- | --- | --- |
| OPERATOR | Human | Someone on-site, an Integrator, or a End-User |
| PRODUCTION | Non-Human |  Production PC user : Production Test Scripts, run by the production PC on End-Of-Line, or by the CI  |
| L3SUPPORT | Human | A developer, such as a tlgate project engineer or prodict owner, providing support or someone acting   |
| CONFIGURATOR | Non-Human |  User used by the COnfiguration Application  |

### Firewall (iptables) rules

TODO

### Dbus allow/deny rules

TODO

### Rescue Console Environment Specification

see SEPASSRFNT-70 :  diag or rescue user can interact in front panel UART, but cannot be root and do anything
### HTTP server configuration

TODO
### Modbus  and IEC10x Security Features

Not supported with SL1, would required to buy an update from Triangle Microworks, for about 40k$.

## References

* Introduction to Embedde Linux Security : https://embeddedbits.org/introduction-embedded-linux-security-part-1/
* IEC-62443 Practicalities : https://download.schneider-electric.com/files?p_enDocType=White+Paper&p_File_Name=998-20186845_GMA-US.pdf&p_Doc_Ref=998-20186845
* CGI bin vulnerabilities: https://www.loginsoft.com/blog/2018/07/25/introduction-to-common-gateway-interface-and-cgi-vulnerabilities/

[Back](toc.md)
