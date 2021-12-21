Test Strategy and Test Plan
===========================

| Technical Notes and Specification | Current [Maturity Grade](SEPASSRFNT-96-development.md)| Comments |
| :---: | :---: | --- |
| (this document)| MG30 (18/12/21) | Specification to review |

# Domain of Application

* This document is part or the Project's Software Development Plan, which is the part of the Quality Manual related to Software Development Processes.
* It contains:
	- Definition of applicable test categories
	- Versioned Test Plans (in this doc for simplicity)
	- Applicable Test Process and Environment

# Applicable Test Categories

## Unit Tests (UT)

### Definition

* Unit Tests are unitary tests of library functions. Depending on the project normative requirement, this can cover all and every methods in the code, to just public API functions, or just leaf functions.

* UT shall prove the **correcness** for the tested functions versus:
	- nominal arguments and return values processing
	- calling context requirements (e.g. thread safe or ASIL/QM etc...)

* UT shall prove the **robustness** for the tested functions versus:
	- out of specification argument values injection: error and failure tree is fully covered and handled
	- wrong calling context (e.g. uninitialized library, thread contention etc...)
	- language boundaries and limitations (e.g. invalid pointers, casting errors etc...)

### Applicable project Rules

* due to overall lack of resources and commitment in the project, it **is not expected that developers will provide unit tests**. However, UT for the **public API functions** is recommended.
* UT can be implemented with either technology:
	- ctest: as part of the cmake metadata
	- gtest: using google test
	- simple suite of shell scripts calling unitary client apps

## Functionnal Tests (FT)

### Description

Functional tests are aimed at proving that a feature is implemented as per its detailed specification.
* In the case of a public library, FT are similar to UT for the public entry points.
* In the case of a sub-system or application, FT is whatever test article is required to check that a specified functionality succeeds and fails as specified.

### Applicable project Rules

**Functionnal Tests are expected** for the following libraries and applications:

* cgi bin
* tlgate-app
* tlgate-eeprom and tlgate-vpd library
* sisgateway
* Tlgate.exe

Ideally, all the **commodities libraries like tmw101, tmw104, tlgate-agn, tlgate-iec10x (etc...) should also be stubbed for FT**, but...

See section "Functional Test Plan" in this document.

## Integration Tests (IT) and Key Performance Indicators (KPI)

Integration Tests are aimed to prove correctness, robustness and performance (CPU load, memory footprint) of the Interfaces of a tested sub-system, in the integrated context.
A minimal implementation of the IT, are FTs for the sub-system.

KPIs are specialized ITs, that measure a required performance item.

See section "Key Performance Indicators" in this document.

## Acceptance Tests (AT)

Acceptance Tests are aimed to assess the versions versus the on-field/customer purpose of the product.
It is not a technical qualification step, but a business/product qualification: is it what the customer initially asked for, and does it make sense for his purpose.

# Applicable Test Process

## Actors

| Actor Name | Description |
| --- | --- |
| MAINTAINER | A person, who is in charge of development of a given software component |
| OPERATOR | A person, who is responsible for the production tests execution, he qualifies a release for Acceptance. |
| DEVELOPER | A person, who contributes by making any code or documentation change to a given software component |
| INTEGRATOR | A person,  who makes versioned system changes to release a given software component into the system |
| PRODUCER | A person, like the Product Owner (agile) or Software Project Manager, who commits with the system version, and produces the formal document bundle of the version |
| QA_MANAGER | A person, who is responsible for the application of the Quality Manual, for the process applicable to the project. |

## From MG40 to MG60


This describes Testing during the steps, from Concept to Validation.

### Who

MG60 is gated OK by the PRODUCER or QA_MANAGER, when the MAINTAINER or a software component can prove that there is a successful FT test report for that component, matching the test plan of the version.
Hence, the MAINTAINER must: 
* review and update the FTP for his component
* implement the FT
* upon "DONE" of the implementation tracking JIRA, attach a successful FT test repport

### how and when

| Trigger | scope | Host-based Tests  Execution | Target-based Test Execution |test types |
| --- | --- | --- | --- | --- |
| a new commit to "dev" branch | one component | run by DEVELOPPER, optional | DEVELOPER uploads to test target, or calls REST API to post matching test to LAVA instance | **UT+FT** |
| a new commit to "master" branch | one component | run by CI, using gitlab | run by CI : calls REST API to post matching test to LAVA instance | **FT** |
| nightly | full system **(DEV IMAGE)** | not applicable | run by CI : CI calls REST API to post matching test to LAVA instance and runs high-level integration test(s) |**FT+IT**|

## From MG60 to MG80


This describes the steps, from Validation to Release.

### Who

MG79 is gated OK by the PRODUCER or QA_MANAGER, when the CI provides a successful Acceptance Test (AT) Report for a given system build.
Hence the INTEGRATOR must:
* create the various meta data in recipes and yocto build config files, to pull the tagged sha1
* tag all the sha1, for the given version

### how and when

| Trigger | scope | Host-based Tests  Execution | Target-based Test Execution | test types |
| --- | --- | --- | --- | --- |
| manual FT | full system **(RELEASE IMAGE)** | not applicable | run by CI: calls REST API to post matching test to LAVA instance and runs high-level integration test(s) | **FT+IT** |
| manual AT | full system **(RELEASE IMAGE)** | not applicable | run by OPERATOR : manual/monkey "tests" to check that modbus, website, tlgate.exe config and sisgsteway app work nominally | **AT** |


# Versionned Test Plans

## Functionnal Test Plan (FTP)

The FTP defines:
* tested component
* repository name of the component
* link to the applicable system requirement (omitted in this project)
* ID og the matching high level feature (HLF) to establish the Tracability Matrix (omitted in this project)
* Location of the test implementation (link to test documentation)
### Pre conditions

pre-conditions are defined per test in the test documentation.
However, for the high-level tests and Integration Tests, it is assumed, that tlgate (tlgate.conf) is matching the factory default test configuration.

### FT Test Plan vor v1.10.0

A **minimal FTP** (feel free to add tests!!) for Sisgateway v6 is as follows:

| Repo | Component | Description | Test ID | Host-based test location | Target-based test location | 
| --- | --- | --- | --- | --- | --- |
| web-cgi | Diagnostique | cgi bin application | FT_CGI_00 | TODO | TODO |
| web-cgi | Etat_Data_Eqt | cgi bin application | FT_CGI_01 |TODO | TODO |
| web-cgi | Etat_Eqt_Tcp | cgi bin application | FT_CGI_02 |TODO | TODO |
| web-cgi | Modif_Eqt_Tcp | cgi bin application | FT_CGI_03 |TODO | TODO |
| web-cgi | Modif_Port | cgi bin application | FT_CGI_04 |TODO | TODO |
| web-cgi | Parametrage | cgi bin application | FT_CGI_05 |TODO | TODO |
| web-cgi | Param_Red_GW | cgi bin application | FT_CGI_06 |TODO | TODO |
| web-cgi | Param_Sync_Eqt | cgi bin application | FT_CGI_07 |TODO | TODO |
| web-cgi | Red_Client | cgi bin application | FT_CGI_08 |TODO | TODO |
| web-cgi | Saisie_Red_Voie | cgi bin application | FT_CGI_09 |TODO | TODO |
| web-cgi | Select_TI | cgi bin application | FT_CGI_10 |TODO | TODO |
| web-cgi | Stat_CL101 | cgi bin application | FT_CGI_11 |TODO | TODO |
| web-cgi | Saisie_Red_Voie | cgi bin application | FT_CGI_12 |TODO | TODO |
| web-cgi | Valid_Donnee | cgi bin application | FT_CGI_13 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | init/exit | FT_VPD_00 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | tlgate_vpd_get_serial_number | FT_VPD_01 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | tlgate_vpd_get_product_type | FT_VPD_01 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | tlgate_vpd_get_hostname | FT_VPD_01 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | tlgate_vpd_get_mac_address | FT_VPD_01 |TODO | TODO |
| tlgate-eeprom | tlgate-eeprom | dump to json and bin | FT_VPD_10 |TODO | TODO |
| tlgate-eeprom | tlgate-eeprom | write from json | FT_VPD_11 |TODO | TODO |
| tlgate-eeprom | tlgate-eeprom | write from bin | FT_VPD_12 |TODO | TODO |
| tlgate-app | tlgate application | IEC101 loopback test | FT_TLG_10 |TODO | TODO |
| tlgate-app | tlgate application | IEC104 polling test | FT_TLG_10 |TODO | TODO |
| appgateway | sisgateway application | TODO | TODO |TODO | TODO |

### FT Test Plan vor v1.0.0

A minimal FTP for Sisgateway v6 is as follows:

| Repo | Component | Description | Test ID | Host-based test location | Target-based test location | 
| --- | --- | --- | --- | --- | --- |
| tlgate-eeprom | libtlgate-vpd | init/exit | FT_VPD_00 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | tlgate_vpd_get_serial_number | FT_VPD_01 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | tlgate_vpd_get_product_type | FT_VPD_01 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | tlgate_vpd_get_hostname | FT_VPD_01 |TODO | TODO |
| tlgate-eeprom | libtlgate-vpd | tlgate_vpd_get_mac_address | FT_VPD_01 |TODO | TODO |
| tlgate-eeprom | tlgate-eeprom | dump to json and bin | FT_VPD_10 |TODO | TODO |
| tlgate-eeprom | tlgate-eeprom | write from json | FT_VPD_11 |TODO | TODO |
| tlgate-eeprom | tlgate-eeprom | write from bin | FT_VPD_12 |TODO | TODO |
| tlgate-app | tlgate application | IEC101 loopback test | FT_TLG_10 |TODO | TODO |
| tlgate-app | tlgate application | IEC104 polling test | FT_TLG_10 |TODO | TODO |


## Key Performance Indicators

### Pre conditions

same pre-conditions than FTP.

### KPIs for v01.10.00

A minimal set of KPIs is as follows:

| KPI Description | Product Requirement | Value | Measurement Method | Test Location | 
| --- | --- | --- | --- | --- |
| Boot time | none (technical KPI for non-reg) | < 20s | capture of console prompt on UART, measurement of time between PORz and boot-prompt availability | TODO |
| Application Setup Time | none (technical KPI for non-reg) | < 60s | IEC104 polling, measurement of time between PORz and first successful polling | TODO |
| system CPU load | none (technical KPI for non-reg) | < 60% | IEC104 polling (TBD) using default factory setup (no sisgateway), using /proc/stat and /proc/status read-out | TODO |
| system memory usage | none (technical KPI for non-reg) | < 50%, and not growing overtime | IEC104 polling (TBD) using default factory setup (no sisgateway), using /proc/stat and /proc/status read-out (vmpages) | TODO |
| tlgate memory usage | none (technical KPI for non-reg) | < xx MB (TBD) and not growing overtime | IEC104 polling (TBD) using default factory setup (no sisgateway), using /proc/{pid}/stat (RSS) | TODO |
| iec101slave/master memory usage | none (technical KPI for non-reg) | < xx MB (TBD) and not growing overtime | IEC104 polling (TBD) using default factory setup (no sisgateway), using /proc/{pid}/stat (RSS) | TODO |
| iec104 memory usage | none (technical KPI for non-reg) | < xx MB (TBD) and not growing overtime | IEC104 polling (TBD) using default factory setup (no sisgateway), using /proc/{pid}/stat (RSS) | TODO |
| Website response time | none (technical KPI for non-reg) | < 2s | refresh time for product status page | TODO |
| Tlgate.exe response time | none (technical KPI for non-reg) | < 2s | refresh time for any Tab | TODO |
| Tlgate.exe firmware update time | none (technical KPI for non-reg) | < 10 minutes | end-to-end firmware update process and progress display | TODO |













