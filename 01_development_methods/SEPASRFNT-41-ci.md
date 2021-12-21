Continuous Integration
=======================


## Feature Status and References

| Technical Notes and Specification | Current [Maturity Grade](../01_development_methods/SEPASSRFNT-96-development.md)| Comments |
| :---: | :---: | --- |
|[SEPASSRFNT-41](https://jira.open-groupe.com/browse/SEPASSRFNT-41) | MG30 (18/12/21) | Concept |

## Introduction

Test types, and Actors are defined in the [Test Strategy](test-strategy.md) document, **please read it first if you have'nt so you share the same terminologies and definitions**.

We wish to use Continous Integration (CI), i.e. triggered build of integration artifact and project deliverables, and test of those deliverables againts Functional Tests (FT) and Integrations Tests (IT).

The CI infrastructure shall allows us to: 
* automate testing based on changes (gitlab commit triggers)
* monitor deliverables quality based on timer triggers (nightly builds)
* improve productivity for developers when testing, byt the use of pipelines (manually triggered build and tests plans)

The CI infrastructure must be simple and fully manageable by MAINTAINERS and DEVELOPERS, and must not required generic IT support to intervene.
It **must be reliable and secured over several several years**, because it is the main tool of release qualification.

**A Software release is essentially the formal bundling and storing of a "good" nightly build, that also passes Acceptance Tests (AT).**



