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

For the various test categories definitions (FT/IT/AT) please first read the [test strategy (Q/A docutment)](test-strategy.md)

# Definitions

## Versions

* We use Semantic Versioning (SEMVER), as normalized here: https://semver.org/
* Versions must be defined beforehand in the project management environment, and consist into a list of HLF/LLFs that makes sense in terms of milestones for technical progress and incremental product qualification. They can be accessed in JIRA from [this location](https://jira.open-groupe.com/projects/SEPASSRFNT?selectedItem=com.atlassian.jira.jira-projects-plugin%3Arelease-page&status=unreleased)

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

## Release Candidates

* A release candidate (RC) is a tag in a given baseline branch, in the form "tlgate-vMM.mm-rc0x".
* Release Candidate (rc) baselines are created based on a "good" nightly build of the baseline, meaning that IT were passed with success.
* a RC is manually created by the Product Owner (software project manager)
* a RC can undergo Acceptance Tests (AT), to be customer-qualified.

Release Candidates are transformed into Releases, if Acceptance Tests pass, by branching.
