# Apigee Simple GitLab Azure Pipeline for Configurable Proxy

[![Generic badge](https://img.shields.io/badge/status-beta-yellow.svg)](https://shields.io/)

**This is not an official Google product.**<BR>This implementation is not an official Google product, nor is it part of an official Google product. Support is available on a best-effort basis via GitLab.

***

## Goal

Simple implementation for a CI/CD pipeline for Apigee Configurable Proxy using GitHub repository, [Azure Pipeline](https://docs.microsoft.com/en-us/azure/devops/pipelines/get-started/what-is-azure-pipelines?view=azure-devops) and gcloud command.

The CICD pipeline includes:

- Integration testing of the deployed proxy using
  [apickli](https://GitLab.com/apickli/apickli)
- Deployment of the API Configurable Proxy bundle using Google gcloud command pre-installed on Microsoft-hosted agents
 
### API Proxy and Apigee configuration

The folder [./proxy](./proxy) includes a simple Configurable API proxy bundle, as well as the following resources:

- [azure-pipelines File](.azure-pipelines.yml) to define an Azure DevOps multi-branch pipeline.
- [test Folder](./test) to hold the integration tests (Apickli).


## Target Audience

- Operations
- API Engineers
- Security


## Limitations & Requirements

- The authentication to the Apigee X management API is done using a GCP Service Account. See [CI/CD Configuration Instructions](#CI/CD-Configuration-Instructions).


## Prerequisites

### Apigee Configurable Environment

This CI/CD pipeline is an example of an Apigee proxy deployment to a configurable environment of an Apigee X organization. <BR>
If you are not familiar with Configurable Environment, please first read Apigee documentation [Configurable API proxies](https://cloud.google.com/apigee/docs/api-platform/develop/configurable-api-proxies).<BR>
If you don't have a Configurable environment, you have first to provision one. See [Provision an Apigee environment](https://cloud.google.com/apigee/docs/api-platform/develop/create-configurable-proxy#provision-an-apigee-environment).


### Azure DevOps

The setup described in this reference implementation is based on Azure DevOps pipeline. So, you must have a Azure account you will use to create a pipeline linked to this GitHub repository. See [Azure DevOps Services](https://azure.microsoft.com/en-us/services/devops/).


## CI/CD Configuration Instructions


### Google Cloud: Create Service Account

Apigee X deployement requires a GCP Service Account with the following roles (or a custom role with all required permissions):

- Apigee Environment Admin

To create it in your Apigee organization's GCP project, use following gcloud commands (or GCP Web UI):

```sh
SA_NAME=<your-new-service-account-name>

gcloud iam service-accounts create $SA_NAME --display-name="GitLab-ci Service Account"

PROJECT_ID=$(gcloud config get-value project)
GitLab_SA=$SA_NAME@$PROJECT_ID.iam.gserviceaccount.com

gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="serviceAccount:$GitLab_SA" \
  --role="roles/apigee.environmentAdmin"

gcloud projects add-iam-policy-binding "$PROJECT_ID" \
  --member="serviceAccount:$GitLab_SA" \
  --role="roles/apigee.apiAdmin"

gcloud iam service-accounts keys create $SA_NAME-key.json --iam-account=$GitLab_SA --key-file-type=json 

```

Copy `<your-new-service-account-name>-key.json` file content to clipboard. 


### Initialize a GitHub Repository

To clone the `Apigee-Configurable-Simple-Azure-Pipeline` in your GitHub repository `github.com/my-user/my-api-proxy-repo`, follow these
steps:

```bash
git clone git@github.com:g-lalevee/Apigee-Configurable-Simple-Azure-Pipeline.git
cd Apigee-Configurable-Simple-Azure-Pipeline
git init
git remote add origin git@github.com:my-user/my-api-proxy.git
git checkout -b feature/cicd-pipeline
git add .
git commit -m "initial commit"
git push -u origin feature/cicd-pipeline
```
 
 ### Azure Pipeline Configuration 

1.  Create a pipeline<BR>
In your [Azure DevOps account](https://dev.azure.com), create a new project. From the **Pipelines** menu, select **Pipeline** and select **GitHub**, then select your cloned repository as source repository. Terminate your pipeline configuration and save it.<BR>
Next step will be to add Apigee credentials to your pipeline. 


2.  Add pipeline variable `GCP_SERVICE_ACCOUNT`, to store your GCP Service Account json key:
- Go to **Pipelines** menu, edit the pipeline, then **Variables** button to add variables.
- Click the **+** button.<BR>In the New variable modal, fill in the details:
  - Key: GCP_SERVICE_ACCOUNT
  - Value: paste clipboard (containing GCP SA JSON key copied before)
  - Keep this value secret: checked
  - Click the **OK** button

3.  (option) Force triggered pipeline execution<BR>If you don't want to manage build trigger from azure-pipelines.yml (see [Azure DevOps Continuous integration triggers](https://docs.microsoft.com/en-us/azure/devops/pipelines/repos/github?view=azure-devops&tabs=yaml#ci-triggers)), you can force it using Pipeline Settings:
- Go to **Pipelines** menu, edit the pipeline, then **More options** button and select **Triggers**.
- In **Continuous Integration** section, check **Override the YAML continuous integration trigger from here** and **Enable continuous integration**



## Run the pipeline

Using your favorite IDE...
1.  Update the **azure-pipelines.yml** file<BR>
    In **Variables** section...
    - change `APIGEE_ORG` value by your target Apigee X organization name
    - change `APIGEE_CONFIG_ENV` value by your target Apigee X configurable environment name
    - change `TEST_HOST` value by the hostname used to call API Proxy once deployed (integration test)
3. Save
4. Commit, Push.. et voila!


Use the Azure DeveOps UI to monitor your pipeline execution and read test reports:

- Go to **Pipelines** menu, select the pipeline, and select the **Runs** section. <BR> Click on the running build you want to monitor.
- The summary page displays builds summary and status. Click on running **Job**
- You can see all steps running, and their logs
- When the build is over, click on the top-left arrow. Then, you see execution status and link Apickli artifacts generated ("1 published")
- Click on artifact link, to access artifact folders and list : apickli_report.html
- Click on artifact apickli_report.html, to download it (you can also click on **More Options** menu to download a zipped folder). Open file with your browser.

---
**NOTE**

The **proxy** folder is the base of your Configurable API proxies file structure, as shown in the following figure:

```sh
.
└── proxy
    └── src
        └── main
            └── apigee
               ├── apiproxies
               |    └── hipster-conf 
               |        └── config.yaml
               └── environments
                    └── ENVIRONMENT-NAME
                        ├── deployment.json
                        └── targetservers.json
```

In this sample, we replaced the folder with the name of the target environment (and containing its configuration) by a variable : it will be automatically replaced by the value of `APIGEE_CONFIG_ENV`

---



