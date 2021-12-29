Release Strategy
================

| Technical Notes and Specification | Current [Maturity Grade](SEPASSRFNT-96-development.md)| Comments |
| :---: | :---: | --- |
| (this document)| MG30 (18/12/21) | Specification to review |

# Introduction

This document provides a set of common definitions: 
* a Version
* a Baseline
* a Release Randidate (RC)
* a Release

This document describes:
* how software is released to the customer to go on-field
* how versions are managed, including versus defects

For the various test categories definitions (FT/IT/AT) please first read the [test strategy (Q/A document)](../01_development_methods/test-strategy.md)

# Definitions

## Versions

* We use **Semantic Versioning (SEMVER)**, as normalized here: https://semver.org/
* Versions must be defined beforehand in the project management environment, and consist into a list of HLF/LLFs that makes sense in terms of **milestones for manageable technical progress and incremental product qualification**. They can be accessed in JIRA from [this location](https://jira.open-groupe.com/projects/SEPASSRFNT?selectedItem=com.atlassian.jira.jira-projects-plugin%3Arelease-page&status=unreleased)

Currently defined versions are: 

| Name | Goals | ETA | Status as of 28/12/21 | 
| --- | --- | --- | --- |
| tlgate-0.99 | baseline for RUN1 validation | December 21 | RELEASED |
| tlgate-1.0 | baseline for tlgate-app integration and validation, and complete flashing process | February 22 | UNDER DEVELOPMENT |
| tlgate-1.1 | baseline for sisgateway integration and validation, and firmware-update process | April 22 | UNDER DEVELOPMENT |
| tlgate-1.10 | baseline for end-to-end product qualification (with windows app) and customer release | July 22 | UNDER DEVELOPMENT |

## Baselines

* **A baseline, is essentially a set of branches, named after a version**, that allows to rebuild that version reliably and identically.
* baselines are protected once released: branches can only be re-opened for modification by the Product Owner
* baselines allow to manage post-release changes, like hot-fixes is any is required.
* An additional baseline is defined, **"develop"** used for default development branch, this is the project's "bleeding edge".

The project uses a **TLGATE_BASELINE yocto variable**, so that we can build for a  given released version, or for *develop*.
* When TLGATE_BASELINE is used, the meta-layers, and all project repositories are checked out, using branch "develop" and HEAD. 
* When using a specific version, for instance "tlgate-v0.99", the branches with the same name are used.

FOr now, in tlgate-v0.99, SRCREV is AUTOREV, but with actual/contractual version, recipes should use:

```
SRC_URI="ssh://git....;branch=$[TLGATE_BASELINE}"
SRCREV="a fixed sha1"
```
## Release Candidates

* **A release candidate (RC) is a tag in a given baseline branch, in the form "tlgate-vMM.mm-rc0x".**
* Release Candidate (rc) baselines are created based on a "good" nightly build of the baseline, meaning that IT were passed with success.
* a RC is manually created by the Product Owner (software project manager)
* a RC can undergo Acceptance Tests (AT), to be customer-qualified.

Release Candidates are transformed into Releases, if Acceptance Tests pass, by branching.

## Release

A release consists into:
* the protected BOOST branches, named after the baseline.
* the build artifacts and deliverables, stored in a reliable location
* the formal documentation bundle

### Branch Protection

every project repository that is involved in building software released to the customer (target software, windows application) must have:
* a "develop" branch, used as default gitlab branch, and were develoments occur.
* one or more other beselines branches

When a version is released, the system integrator (or user called "PRODUCER" in the test-strategy doc) must:

* create a branch with the version name e.g. "tlgate-v1.00", "tlgate-v1.01", "tlgate-v1.10" for all project repositories
* in gitlab, this branch must be protected, so that no-one can push or delete from it, unless the PRODUCER needs a hotfix added.
* a configuration file in the [yocto setup repository](https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/opengrp-gateway-sdk/-/tree/develop/configs) must be created, e.g. "opengrp-gateway-tlgate-v1.00.txt", with the matching TLGATE_BASELINE value.

Hot-fixes can be done to accommodate the customer or end-user, in case a complete version upgrade is considered risky, in the case, a bugfix commit can be back ported from the "bleeding edge" in "develop", to another baseline.

### Artifacts and Deliverables


* Artifacts are end-products of the build and qualification process, that must be stored for a given version
* Deliverables are end-products that are provided to the customer directly : downloadable update bundle, documentation etc..

| Item Type | Item Name | Storage Location | File Type |
| --- | --- | --- | --- |
| ARTIFACT | application flash image | TBD | .WIC (raw image with partitioning) |
| ARTIFACT | maintenance flash image | TBD |.WIC (raw image with partitioning) |
| DELIVERABLE | RAUC or sw-update update bundle | TBD | .rauc |

To Be Defined: storage location should be suitable for long-term/secured storage and link sharing, ideally something like SharePoint of Artifatory.

### Formal Documentation Bundle

| Item Type | Item Name | Storage Location | File Type |
| --- | --- | --- | --- |
| DELIVERABLE | Release Note | TBD | .pdf |
| DELIVERABLE | AT test report | TBD | .pdf |
| DELIVERABLE | User Manual | TBD | .pdf |
