# Demo for the IaC Compliance Management Framework (IACMF)

This repo allows setting up and running a case study that demonstrates the usage of IACMF to check and enforce a compliance rule on a cloud application during runtime.
In summary, the rule demands that no unexpected docker containers are allowed to run within the cloud deployment.

This and other case studies are also demonstrated in [this video](https://clipchamp.com/watch/Geqe70aPkjh).

## Prerequisites

- Windows 11 (Only tested on this version. Other OS types and versions might also be fine.)
- Google Chrome or Microsoft Edge browsers (only tested on these browsers.)
- Docker Desktop (tested on v4.19.0)
- __Stable__ Internet access.
- `git`
- Sufficient disk space (the needed docker images are around 7 GB)
- __At the beginning of the experiment, no docker containers are allowed to be running on the local docker engine.__
- The following ports must be free:
    - 80
    - 1337
    - 1883
    - 2222
    - 4200
    - 4406
    - 7070
    - 8080 - 8088
    - 8092
    - 8098 - 8099
    - 9091
    - 9763
    - 13373


## The Overall Experiment Design

IACMF consists of a [backend](https://github.com/IAC2-Project/iacmf) Java SpringBoot Application and an Angular [frontend](https://github.com/IAC2-Project/iacmf-ui).
Additionally, IACMF requires [Eclipse Winery](https://github.com/eclipse/winery) to design compliance rules and visualize reconstructed cloud application instances.
It also requires a Mysql database to store its state, e.g., compliance jobs, compliance job executions, etc.

In this experiment, we deploy and provision a [TOSCA](https://www.oasis-open.org/committees/tc_home.php?wg_abbrev=tosca) cloud application using [OpenTOSCA Container](https://github.com/OpenTOSCA/container) (an existing TOSCA engine).
The test cloud app is a three-tier application deployed on three different docker containers.
IACMF is supposed to manage the runtime IaC compliance of this application.
In this example, only one compliance rule is applicable, which states that no unexpected docker containers are allowed to run on the application's infrastructure. Of course, the framework is capable of running multiple compliance rules and managing the compliance of multiple cloud applications simultaneously.

We show that the application is compliant after provisioning, and then we introduce a violation during runtime and show that IACMF detects it and fixes it automatically.

## The Experiment

This section describes the steps to reproduce the experiment.

### Estimated Duration

__Setup__: ~20 minutes

__Experiment__: ~10 minutes


### Overall Steps

In the following, the high-level steps of the experiment are presented.
Certain steps have more details, which are presented in separate sections below.

1. Clone this repository to a local folder. In a command line prompt, run:
    ```
    git clone https://github.com/IAC2-Project/case-study.git
    ```
   And navigate to the created folder.

2. Find the public IPV4 address of your machine (e.g.,  use __cmd__ or __PowerShell__ and run the `ipconfig /all` command). For example, if you are connected to the internet using WiFi, the corresponding entry would be called _"Wireless-Lan-Adapter WLAN"_ or similar to that.
Copy the IPV4 value to the variable called
   `PUBLIC_HOSTNAME` inside the file `.env`, and save the file (_this is necessary to allow docker containers
   to correctly communicate with each other_).

3. Pull the latest version of the necessary docker images by opening the command line inside the cloned folder and running:
    ```
    docker-compose pull
    ```

4. Build the remaining docker images (that represent the frontend and the backend) by running the following command (may take up to 10 minutes):
    ```
    docker-compose build --no-cache
    ```

5. Start the docker containers by running the following command: 
    ```
    docker-compose up -d
    ```

6. Wait for around 2 minutes for the containers and the hosted application to finish starting.
   The docker container for the OpenTOSCA Container (called `container-1`) and for the IACMF Backend (called `iacmf-backend-1`) are the longest to finish booting up.
   __Note that errors related to not being able to run certain `DDL` SQL commands can be ignored.__
   
   The following screenshots from the Docker Desktop application show the corresponding logs when booting is finished:
   
   __Hint:__ You can also run the contains without detached mode, so you can see the logs in the terminal, i.e., `docker compose up`
   
   ![](./assets/screenshots/OpenTOSCA%20Container%20finished%20startup.png)
   ![](./assets/screenshots/IACMF%20Backend%20finished%20startup.png)

7. Deploy a sample TOSCA application on OpenTOSCA Container ([see below for details](#step-7---deploy-a-sample-tosca-application-on-opentosca-container)).

8. Provision an instance of the cloud application ([see below for more details](#step-8---provision-an-instance-of-the-cloud-application)).

9. Set up a new compliance job in IACMF and try it ([see below for more details](#step-9---setup-a-new-compliance-job-in-iacmf-and-try-it)).

10. Introduce a compliance violation in the cloud application instance ([see below for more details](#step-10---introduce-a-compliance-violation-in-the-cloud-application-instnace)).

11. Run an execution of the compliance job to detect and automatically fix the violation ([see below for more details](#step-11---run-an-execution-of-the-complaince-job-to-detect-and-automatically-fix-the-violation)).

Steps 9-11 represent the actual experiments.
Steps 1-8 represent the setup phase.



### Step 7 - Deploy a Sample TOSCA Application on OpenTOSCA Container

1. Obtain the archive file that will be used to deploy the cloud application from the
   [Zenodo repository](https://doi.org/10.5281/zenodo.10069931) hosting a copy of this case study
   (the file is also available in a [release within this Github repository](https://github.com/IAC2-Project/case-study/releases/tag/v1.0.0)).
   The file is called: `RealWorld-Application_Angular-Spring-MySQL-w2-wip1.csar`, and it is around 150 MB.

2. Open the browser on __`http://<YOUR_IP_ADDRESS>:8088`__ to view the management UI of OpenTOSCA Container.

3. Click on the __Administration__ tab, and make sure that the property `API ENDPOINT` has the value: `http://<YOUR_IP_ADDRESS>:1337`
 ![](./assets/screenshots/OTContainer%20Admin.png)

4. Go back to the __Applications__ tab and click on the `Upload new Application` button.
   ![](./assets/screenshots/OTContainer%20Upload%20CSAR.png)

5. In the shown dialog, select the obtained `RealWorld-Application_Angular-Spring-MySQL-w2-wip1.csar` file.
   ![](./assets/screenshots/OTContainer%20CSAR%20selected.png)

6. Click on `Upload` (the upload may take around 1 minute).
   ![](./assets/screenshots/OTContainer%20CSAR%20Uploaded.png)

[[go to overview]](#overall-steps)



### Step 8 - Provision an Instance of the Cloud Application

1. Click on the `Show details...` button (blue button within the deployed application's card).

2. Click on the plus button to start the process of provisioning a new instance of the cloud application.
   ![](./assets/screenshots/OTContainer%20Provision%20Instance.png)

3. Select the `initiate` lifecycle operation.
   ![](./assets/screenshots/OTContainer%20select%20initiate%20plan.png)

4. Click on the `Next` button.

5. Fill up the arguments needed to provision an app instance as follows:
    - __BackendPort__: `13373`
    - __DockerUrl__: `tcp://<YOUR_IP_ADDRESS>:2222`.
    - __FrontendPort__: `80`
    ![](./assets//screenshots/OTContainer%20select%20configure%20instance.png)

6. Click on `Run`.

7. Wait until the application is provisioned.
   This may take around 7 minutes.
   ![](./assets/screenshots/OTContainer%20Provision%20Instance-CREATING.png)

8. Click on the refresh button to refresh the state. 
   If the state shows `CREATED`, then the provisioning is finished successfully.
   ![](./assets/screenshots/)

9. Copy or note down the value of the `Instance ID`, it will be needed when setting up the compliance job.

10. You can access the demo-application at <http://localhost>.

[[go to overview]](#overall-steps)




### Step 9 - Set Up a New Compliance Job in IACMF and Try It



#### Create a Production System

A _Production System_ is an internal representation within IACMF about a cloud application instance that needs to be checked for compliance.
The following steps create a Production System for the previously instantiated cloud application (steps 7 & 8).

1. Open the UI of IACMF in the browser: http://localhost:4200/
   ![](./assets/screenshots/Experiment/IACMF%20start%20page.png)

2. Click on `Add Production System`.

3. In the popup click on `Create Test Data` to speed up the value entry process.
   ![](./assets/screenshots/Experiment/IACMF%20Create%20Test%20Data%20Button.png)

4. In the section called `Production System Properties` fill in the value of the property called `opentoscacontainer_instanceId`.
   ![](./assets/screenshots/Experiment/IACMF%20change%20instance%20id.png)

5. Click `OK`. The Production System is added.
   ![](./assets/screenshots/Experiment/IACMF%20Production%20System%20added.png)



#### Create a Compliance Rule

A _Compliance Rule_ is an internal representation within IACMF about a technical compliance rule that is defined in an external tool, e.g., Eclipse Winery.
We will now create a _Compliance Rule_ that represents an already-created technical compliance rule that can be viewed and accessed under the following link:
http://localhost:8080/#/compliancerules/http%253A%252F%252Fwww.example.org%252Ftosca%252Fcompliancerules/no-unexpected-docker-containers_w1-wip1/readme

1. In IACMF, go to the `Compliance Rules` tab.

2. Click on `Add Compliance Rule`.
   ![](./assets/screenshots/Experiment/IACMF%20Add%20New%20Compliance%20Rule.png)

3. Click on `Create Test Data` to speed up the data entry process.

4. Click on `OK`. Now the rule is created.
   ![](./assets/screenshots/Experiment/IACMF%20Compliance%20Rule%20added.png)


#### Create a Compliance Job

A _Compliance Job_ represents the mapping between a Production System and a set of Compliance Rules applicable to it.
Additionally, information about how to reconstruct the architecture of the Production System, how to check the Compliance Rule,
how to fix violations, how to validate the system after fixes, and how to report compliance job executions are also provided
via a Compliance Job.

1. In IACMF, go to the `Compliance Jobs` tab.

2. Click on `Add Compliance Job`.

   ![](./assets/screenshots/Experiment/IACMF%20Add%20Compliance%20Job.png)

3. In step "Basic Information", choose a name for the Compliance Job and click `Next`.

4. In step "Production System", select the Production System created earlier. Click `Next`.

   ![](./assets/screenshots/Experiment/IACMF%20CJ%20Select%20Production%20System.png)

5. In step "Refinement", select the `docker-refinement-plugin` and click `Add`.
   Click `Next`.
   ![](./assets/screenshots/Experiment/IACMF%20CJ%20Select%20&%20Add%20Docker%20Refinement%20Plugin.png)

6. In step "Compliance Rules", select the compliance rule created earlier, and click on `Add`.
   ![](./assets/screenshots/Experiment/IACMF%20CJ%20Select%20&%20Add%20CR.png)

7. While in the same step, click on `Configure`.

8. In the popup, click on `Create Test Data` to speed up the data entry process.
   Update the value of __ENGINE_URL__ to: `tcp://<YOUR_IP_ADDRESS>:2222` to ensure that the correct Docker Engine will be used.
   ![](./assets/screenshots/Experiment/IACMF%20CJ%20Configure%20Compliance%20Rule.png)

9. Click on `OK`.
   ![](./assets/screenshots/Experiment/IACMF%20CJ%20CR%20configured.png)

10. Click on `Next` (you might need to scroll down a little).

11. In step "Checking", select the `subgraph-matching-checking-plugin` and click `Next`.
    ![](./assets/screenshots/Experiment/IACMF%20CJ%20Select%20Issue%20Checking%20Plugin.png)

12. In step "Compliance Issue Fixing", select a mapping between the issue type `UNEXPECTED_DOCKER_CONTAINERS` and the
    `docker-container-issue-fixing-plugin` and click on `Create Fixing Configuration`.
    Click `Next`.
    ![](./assets/screenshots/Experiment/IACMF%20CJ%20Configure%20Issue%20Fixing.png)

13. In step "Validation", select and add the `NOOP` plugin, and click `Next`.
    ![](./assets/screenshots/Experiment/IACMF%20CJ%20Select%20&%20Add%20Validation.png)

14. In step "Reporting", select the `no-ops-reporting-plugin` and click `Done`.
    ![](./assets/screenshots/Experiment/IACMF%20CJ%20Select%20Reporting%20Plugin.png)

    The compliance job is now configured.
    ![](./assets/screenshots/Experiment/IACMF%20CJ%20created.png)

15. To try the compliance job, click on `Start Execution`
    ![](./assets/screenshots/Experiment/IACMF%20CJ%20running%20-%20no%20violation.png)

16. After running for a few seconds, the execution finishes without reporting any errors
    (because the cloud application instance is still compliant).
    ![](./assets/screenshots/Experiment/IACMF%20CJ%20finished%20-%20no%20violation.png)

17. (Optional) In the section __Instnace Model__, click on the link _Click here to visualize_.
    This link sends a copy of the instance model that IACMF has reconstructed to Eclipse Winery to visualize it,
    which is opened in a new tab (the button `Layout` at the top helps enhance the layout of the graph):
    ![](./assets/screenshots/Experiment/Winery%20Instance%20Model%20No%20Violations%201.png)

This finishes the process of setting up a compliance job and testing it.

[[go to overview]](#overall-steps)



### Step 10 - Introduce a Compliance Violation in the Cloud Application Instnace

In this step, we will introduce a new Docker container running on the same Docker engine as the cloud application, which constitutes a compliance rule violation.

1. Open a __git bash__ command prompt.

2. Run the following commands:
   ```
   docker exec iac2-dind-1 docker pull bash
   winpty docker exec -it iac2-dind-1 docker run -it bash
   ```

3. __Do not close the command prompt window!__

[[go to overview]](#overall-steps)



### Step 11 - Run an Execution of the Compliance Job to Detect and Automatically Fix the Violation

1. Go back to the tab where IACMF UI is shown.

2. Navigate to the `Compliance Jobs` tab.

3. Start a new execution of the existing compliance job by clicking on the `Start Execution` button.
   ![](./assets/screenshots/Experiment/IACMF%20CJ%20running%20-%20%20violation.png)
4. See the state of the execution change.
   After around a minute the execution will be finished with the State "Finished Normally. Violations detected, and all are fixed".
   ![](./assets/screenshots/Experiment/IACMF%20CJ%20finished%20-%20violation.png)

5. You can click on the detected violation to see more details about the violation and how it was fixed.

6. (Optional) visualize the new instance model in Winery by clicking on the link _Click here to visualize_,
   and notice the new, unexpected Docker container.
   ![](./assets/screenshots/Experiment/Winery%20Instance%20Model%20Violations.png)
7. (Optional) execute the compliance job one more time, and notice that the cloud application is compliant again.
   ![](./assets/screenshots/Experiment/IACMF%20CJ%20finished%20-%20no%20violation%202.png)

[[go to overview]](#overall-steps)



## License

This work is license under the [Apache 2.0 license](./LICENSE).

SPDX-License-Identifier: `Apache-2.0`
