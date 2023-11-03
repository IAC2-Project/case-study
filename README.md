# Demo for the IaC Compliance Management Framework (IACMF)

This repo allows setting up and running a case study that demonstrates the usage of IACMF to check and enforce a compliance rule on a cloud application during runtime.
In summary, the rule demands that no unexpected docker containers are allowed to run within the cloud deployment.

This and other case studies are also demonstrated in [this video](https://clipchamp.com/watch/Geqe70aPkjh).

## Prerequisits

- Windows 11 (Only tested on this version. Other OS types and versions might also be fine.)
- Docker Desktop (v4.19.0)
- `git`
- Sufficient disk space (the needed docker images are around 7 GB)
- At the beginning of the experiment, no docker containers can be running on the local docker engine.
- The following ports must be free:
    - 80
    - 1337
    - 1883
    - 3306
    - 4200
    - 4406
    - 7070
    - 8080
    - 8081
    - 8082
    - 8083
    - 8085
    - 8086
    - 8087
    - 8088
    - 8092
    - 8099
    - 8098
    - 9763
    - 9091
    - 13373


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
```git clone https://github.com/IAC2-Project/case-study.git```
and navigate to the created folder.
2. Find the public IP address of your machine (e.g., using `ipconfig \all`) and copy it to the variable called `PUBLIC_HOSTNAME` inside the file `.env` (this is necessary to allow docker containers to correctly communicate with each other).
3. Pull the latest version of the necessary docker images by opening the command line inside the cloned folder and running:
``` docker-compose pull```.
4. Build the remaining docker images (that represent the frontend and the backend) by running the following command: ```docker-compose build --no-cache``` (may take up to 10 minutes).
5. Start the docker containers by running the following command: `docker-compose up -d`.
6. Wait for around 2 minutes for the containers and the hosted application to finish starting.
The docker container for the OpenTOSCA Container (called `container-1`) and for the IACMF Backend (called `iacmf-backend-1`) are the longest to finish booting up.
The following screenshots shows their logs when the boot is finished: ![](./assets/screenshots/OpenTOSCA%20Container%20finished%20startup.png)
![](./assets/screenshots/IACMF%20Backend%20finished%20startup.png)
7. Deploy a sample TOSCA application on OpenTOSCA Contaner ([see below for details](#step-7---deploy-a-sample-tosca-application-on-opentosca-container)).
8. Provision an instance of the cloud application ([see below for more details](#step-8---provision-an-instance-of-the-cloud-application)).
9. Setup a new compliance job in IACMF and try it(see below for more details).
10. Introduce a compliance violation in the cloud application instnace (see below for more details).
11. Run an execution of the complaince job to detect and automatically fix the violation (see below for more details).

Steps 9-11 represent the actual experiments. 
Steps 1-8 represent the setup phase.

### Step 7 - Deploy a Sample TOSCA Application on OpenTOSCA Container

1. Obtain the archive file that will be used to deploy the cloud application from the Zenodo repository hosting a copy of this case study (the file is also available in a release within this Github repository).
The file is called: `RealWorld-Application_Angular-Spring-MySQL-w2-wip1.csar`, and it is around 150 MB.
2. Open the browser to http://localhost:8088 view the management UI of OpenTOSCA Container.
3. Click on the `Upload new Application` button.
![](./assets/screenshots/OTContainer%20Upload%20CSAR.png)
4. In the shown dialog, select the obtained `RealWorld-Application_Angular-Spring-MySQL-w2-wip1.csar` file.
![](./assets/screenshots/OTContainer%20CSAR%20selected.png)
5. Click on `Upload` (the upload may take around 1 minute).
![](./assets/screenshots/OTContainer%20CSAR%20Uploaded.png)

[[go to overview]](#overall-steps)

### Step 8 - Provision an Instance of the Cloud Application




