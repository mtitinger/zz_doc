Continuous Integration
=======================


## Feature Status and References

| Technical Notes and Specification | Current [Maturity Grade](../01_development_methods/SEPASSRFNT-96-development.md)| Comments |
| :---: | :---: | --- |
|[SEPASSRFNT-41](https://jira.open-groupe.com/browse/SEPASSRFNT-41) | MG30 (18/12/21) | Concept |

## Introduction

Test types, and Actors are defined in the [**Test Strategy**](test-strategy.md) document, **please read it first if you haven't, so you share the same terminologies and definitions**.

We wish to use Continous Integration (CI), i.e. triggered build of integration artifact and project deliverables, and test of those deliverables againts Functional Tests (FT) and Integrations Tests (IT).

The CI infrastructure shall allows us to: 
* automate testing based on changes (gitlab commit triggers)
* monitor deliverables quality based on timer triggers (nightly builds)
* improve productivity for developers when testing, byt the use of pipelines (manually triggered build and tests plans)

The CI infrastructure must be simple and fully manageable by MAINTAINERS and DEVELOPERS, and must not required generic IT support to intervene.
It **must be reliable and secured over several years**, because it is the main tool of release qualification.

**A Software release is essentially the formal bundling and storing of a "good" nightly build, that also passes Acceptance Tests (AT).**

## CI Infrastrute Functions

* A "Build Plan" is a series of Steps, leading to the execution of any combinaation of the functions described in the following Sections.
* A "Trigger" is a "reasons" or event leading to the execution of a Build Plan.

The CI Infra must allow:
* Time triggers : Build Plan starts at a given hours.
* Manual triggers : Build Plan start when user requests it
* Cross Repository triggers : Build Plan starts whenever changes occur on a different repo in a specified branch.

### Build
#### Images and Deliverables

| Image | Description | Triggers |
| --- | --- | --- |
| opengrp-gateway-img-dev | development image, used for nightly builds | nightly |
| opengrp-gateway-img-maint | maintenance image, used for production and rescue | when any related repo is changed, OR when PRODUCER wishes to release a new version |
| opengrp-gateway-img-emc-tests | specialized image, for EMC-tests | emc-testsuite repo is modified |
| opengrp-gateway-img-release | release image | maual : when PRODUCER decides to release an image, based on a (FT/IT)-qualified development image |

#### Software components

repo : all of the repositories under https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp
trigger: any change to branch master, or manually if user wants to run UT/FT/IT tests.

#### Project Documentation

repo under https://gitlab.boost.open.global/schneider-electric/passerelle_refonte/Software/bsp/sisgateway-internal-documentation
trigger: any change to branch master, or manually if user wants to run UT/FT/IT tests, or upon release.

### Deploy/Store Artifacts

### Schedule Tests

### Collect Results

### Dispatch Tests





