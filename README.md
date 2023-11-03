# Demo for the IaC Compliance Management Framework (IACMF)

This repo allows setting up and running a case study that demonstrates the usage of IACMF to check and enforce a compliance rule on a cloud application during runtime.
In summary, the rule demands that no unexpected docker containers are allowed to run within the cloud deployment.

This and other case studies are also demonstrated in [this video](https://clipchamp.com/watch/Geqe70aPkjh).

## Prerequisits

- Windows 11 (Only tested on this version. Other OS types and versions might also be fine.)
- Docker Desktop (v4.19.0)
- `git`
- Sufficient disk space (the needed docker images are around 7 GB)

## The Overall Experiment Design

IACMF constitutes of a [backend](https://github.com/IAC2-Project/iacmf) Java SpringBoot Application and an Angular [frontend](https://github.com/IAC2-Project/iacmf-ui).
Additionally, IACMF requires [Eclipse Winery](https://github.com/eclipse/winery) to design compliance rules and visialize reconstructed cloud application instances.
It also requires a Mysql database to store its state, e.g., compliance jobs, compliance job executions, etc.

In this experiment, we deploy and provision a [TOSCA](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=tosca) cloud application using [OpenTOSCA Container](https://github.com/OpenTOSCA/container) (an existing TOSCA engine).
The app is a three-tier application deployed on three different docker containers.
IACMF is supposed to manage the runtime IaC compliance of this application.
In this example, only one compliance rule is applicable, which states that no unexpected docker containers are allowed to run on the application's infrastructure. Of course, the framework is capable of running multiple compliance rules and managing the compliance of multiple cloud applications simultaneously.

We show that the application is compliant after provisioning, and then we introduce a violation during runtime and show that IACMF detects it and fixes it automatically.

## The Experiment

### Estimated Duration

__Setup__: ~20 minutes

__Experiment__: ~10 minutes

### Overall Steps

In the following, the high-level steps of the experiment are presented.
Certain steps have more details, which are presented in separate sections below.

1. Clone this repository to a local folder
```git clone ```
