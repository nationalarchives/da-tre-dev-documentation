# Release cycle

Releases are almost always a mixture between human and automated processes. The release cycle can be reduced through automation, especially automation of the testing and deployment processes.

## Releasing infrastructure changes

Infrastructure resources are provisioned and maintained using [Terraform](https://www.terraform.io/intro); there are two GitHub repositories used for the Infrastructure-as-Code (IaC) implementation:

- Environments repository: https://github.com/nationalarchives/da-transform-terraform-environments 
    - This repository contains the configuration for setting up the infrastructure using modules contained in the modules repository.
    - The repository has got branches to mirror the different environments required (DEV, TEST, INT, STAGING and PROD)
- Modules repository: https://github.com/nationalarchives/da-transform-terraform-modules
    - This repository contains the reusable modules, across environments that are used to provision the resources which are referenced by the environments Terraform repository.
    - The repository has got branches to mirror the different environments required (DEV, TEST, INT, STAGING and PROD)

Automation is guaranteed by using a [pipeline for each environment](../dev-deploy-guide/README.md#code-pipelines) in the AWS management account (for the TRE AWS accounts see this [link](../../beta-mvp-architecture/README.md#aws-accounts-management)), which deploys the infrastructure changes to the different environments.

The release cycle for infrastructure changes consists of the steps described in the section [Application Update Process](./../dev-deploy-guide/README.md#application-update-process)

### Releasing new versions of the Step Functions

TO DO

## Releasing new versions of the Parser

TO DO

## Releasing new versions of docker images

AWS Step Function is used to implement multiple workflows to process data from external services and ingest it to other external services. 
A Step Function consists of a sequence of tasks and gateways, tasks are implemented using Lambda funtions packaged as Docker images.

When releasing a new version of a docker image for a Lambda function, the release cycle consists of the following steps:

![pic1](./diagrams/TRE-lamba-deployment.png)

1. a developer raises a new pull request to merge the changes to the `main` branch of GitHub repository [da-transform-judgments-pipeline](https://github.com/nationalarchives/da-transform-judgments-pipeline)
2. 


