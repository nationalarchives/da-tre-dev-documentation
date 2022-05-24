# Development Deployment Guide

The following sections provide a system overview and update instructions:

* [Application Summary](#application-summary)
* [Deployment Environments](#deployment-environments)
* [Code Repositories](#code-repositories)
* [Automation Processes](#automation-processes)
  * [Parameter Store](#parameter-store)
  * [Code Pipelines](#code-pipelines)
* [Application Update Process](#application-update-process)

# Application Summary

The Judgments Pipeline workflow is implemented as a Step Function with Lambda
Function calls to perform operations not covered by built-in tasks.

# Deployment Environments

The application is deployed across the following account and environment
hierarchy:

* Account: **main**
  * Account: **management** (code pipelines, root ECR instance, parameter store)
    * Account: **non-prod** (local ECR instance)
      * Environment: `dev`
      * Environment: `test`
      * Environment: `int`
    * Account: **prod** (local ECR instance)
      * Environment: `staging`
      * Environment: `prod`

Environment resources are created using IaC (Terraform) and have their names
prefixed with that of the environment; for example, a Step Function called
`tre-state-machine` would be deployed as the following entities:

* Account: **non-prod**:
    * `dev-tre-state-machine`
    * `test-tre-state-machine`
    * `int-tre-state-machine`
* Account: **prod**:
    * `staging-tre-state-machine`
    * `prod-tre-state-machine`

The **management** account is used to host:

* Code pipeline automation processes
* A root ECR instance that replicates Docker images to the **non-prod** and
    **prod** accounts
* Parameter Store with per-environment settings

# Code Repositories

| Repository                                                                                                           | Purpose                                                                   | Key Branches                    |
| -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- | ------------------------------- |
| [da-transform-judgments-pipeline](https://github.com/nationalarchives/da-transform-judgments-pipeline/tree/develop)  | Code for Lambda Function Docker images                                    | `develop`                       |
| [da-transform-terraform-modules](https://github.com/nationalarchives/da-transform-terraform-modules)                 | IaC application entities (e.g. Step Function, Lambda Functions, SQS, ...) | `dev`, `int`, `staging`, `prod` |
| [da-transform-terraform-environments](https://github.com/nationalarchives/da-transform-terraform-environments)       | IaC execution entrypoint; IaC system entities                             | `dev`, `int`, `staging`, `prod` |
| [tna-judgments-parser](https://github.com/nationalarchives/tna-judgments-parser) <sup>*</sup>                        | Parser code for Step Function Lambda Docker image                         | `main`                          |

> <sup>*</sup>A Docker image is built by this project when the external
    `tna-judgments-parser` project has a new tag pushed (see `parser-pipeline`
    in [Code Pipelines](#code-pipelines))

# Automation Processes

## Parameter Store

The following Parameter Store entries are used to define properties (such as
ARNs and Step Function Lambda Function versions) that are referenced by
Terraform IaC scripts<sup>*</sup>:

| Parameter        | Purpose                                            |
| ---------------- | -------------------------------------------------- |
| `dev-tfvars`     | Configuration parameters for `dev` environment     |
| `test-tfvars`    | Configuration parameters for `test` environment    |
| `int-tfvars`     | Configuration parameters for `int` environment     |
| `staging-tfvars` | Configuration parameters for `staging` environment |
| `prod-tfvars`    | Configuration parameters for `prod` environment    |

> <sup>*</sup>A code pipeline run must be triggered for Terraform to `apply`
    any change

## Code Pipelines

| Process             | Purpose                                                | Automation Trigger                                                        |
| ------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------- |
| `parser-pipeline`   | Creates Parser ECR Docker images                       | `tna-judgments-parser` tag                                                |
| `terraform-dev`     | Deploys `dev` branch IaC configuration<sup>*</sup>     | `da-transform-terraform-environments` `dev` branch commit<sup>*</sup>     |
| `terraform-test`    | Deploys `test` branch IaC configuration<sup>*</sup>    | `da-transform-terraform-environments` `test` branch commit<sup>*</sup>     |
| `terraform-int`     | Deploys `int` branch IaC configuration<sup>*</sup>     | `da-transform-terraform-environments` `int` branch commit<sup>*</sup>     |
| `terraform-staging` | Deploys `staging` branch IaC configuration<sup>*</sup> | `da-transform-terraform-environments` `staging` branch commit<sup>*</sup> |
| `terraform-prod`    | Deploys `prod` branch IaC configuration<sup>*</sup>    | `da-transform-terraform-environments` `prod` branch commit<sup>*</sup>    |

> <sup>*</sup>The IaC scripts in the `da-transform-terraform-environments`
    repository fetch from the corresponding branch in the
    `da-transform-terraform-modules` repository

> IaC code entrypoint that creates the above processes: [codepipeline.tf](https://github.com/nationalarchives/da-transform-terraform-environments/blob/dev/common/codepipeline.tf)

# Application Update Process

This section describes how to apply updates to the application environments.

The process is:

```
                                              Environments

                                 +-----+
+-- 1 --> feature/fix --+-- 2 -->| dev |
|            branch     |        +-----+
|                       |
|                       |        +------+   +-----+   +---------+   +------+
|                       +-- 3 -->| test |-->| int |-->| staging |-->| prod |
|                                +------+   +-----+   +---------+   +------+
|                                   |
+-----------------------------------+

                                 \___ non-prod ___/   \_______ prod _______/
                                       account                account
```

Some changes may require all [Code Repositories](#code-repositories) be
updated, some may not; adjust the following steps accordingly:

1. Make changes in the `dev` environment:

    1. Create feature branches where required in the following repositories:

        | Repository                            | Source Branch | Example                                                                     |
        | ------------------------------------- | ------------- | --------------------------------------------------------------------------- |
        | `da-transform-judgments-pipeline`     | `develop`     | Step Function Lambda change                                                 |
        | `da-transform-terraform-modules`      | `dev`         | Step Function workflow change<br>Application specific cloud resource change |
        | `da-transform-terraform-environments` | `dev`         | Terraform IaC trigger<sup>*</sup><br>System specific cloud resource change  |

        > <sup>*</sup> The `da-transform-terraform-environments` repository is
            used to trigger IaC Terraform pipelines

    2. Make Step Function updates with the Step Function UI

    3. Copy the Step Function JSON configuration changes from the UI into the
        Step Function's [IaC definition](https://github.com/nationalarchives/da-transform-terraform-modules/blob/dev/step_function/templates/step-function-definition.json.tftpl)

        > Ensure Terraform variables are used and retained where necessary to
            maintain environment separation

    4. Make Lambda Function updates in the `da-transform-judgments-pipeline`
        feature branch

    5. Build and push Lambda Function Docker image(s) to ECR:

        1. Ensure the correct version is set for the Lambda Function
        
            > For `tre-editorial-integration` this is: [`tre-editorial-integration/version.sh`](https://github.com/nationalarchives/da-transform-judgments-pipeline/blob/develop/lambda_functions/tre-editorial-integration/version.sh)

        2. Run the Lambda Function's build script

            > For some images this is: [`lambda_functions/build.sh`](https://github.com/nationalarchives/da-transform-judgments-pipeline/blob/develop/lambda_functions/build.sh)

    6. While the Lambda Function version(s) can be updated manually in the
        console for development testing  purposes, ensure the steps in the
        next section are applied to make the version changes persistent (or
        the next Terraform IaC run will overwrite your change)

2. Test IaC changes in the `dev` environment:

    1. Update the Lambda Function version(s) in `dev-tfvars` in Parameter Store<sup>1</sup>

    2. Merge any Lambda Function updates in `da-transform-judgments-pipeline`
        to the `develop` branch

    3. Merge any Step Function updates in `da-transform-terraform-modules`
        to the `dev` branch

    4. Trigger a Terraform apply by merging the feature branch changes in
        `da-transform-terraform-environments` to the `dev` branch<sup>2</sup>

2. Deploy feature changes into the `test` environment:

    1. Update the Lambda Function version(s) in `test-tfvars` in Parameter Store<sup>1</sup>

    2. Merge any Lambda Function updates in `da-transform-judgments-pipeline`
        to the `test` branch

    3. Merge any Step Function updates in `da-transform-terraform-modules`
        to the `dev` branch

    4. Trigger a Terraform apply by merging the feature branch changes in
        `da-transform-terraform-environments` to the `test` branch<sup>2</sup>

3. Promote changes to the `int` environment:

    1. Update the Lambda Function version(s) in `int-tfvars` in Parameter Store<sup>1</sup>

    2. Merge any Lambda Function updates in `da-transform-judgments-pipeline`
        to the `int` branch

    3. Merge any Step Function updates in `da-transform-terraform-modules`
        to the `int` branch

    4. Trigger a Terraform apply by merging the feature branch changes in
        `da-transform-terraform-environments` to the `int` branch<sup>2</sup>

4. Promote changes to the `staging` environment:

    1. Update the Lambda Function version(s) in `staging-tfvars` in Parameter Store<sup>1</sup>

    2. Merge any Lambda Function updates in `da-transform-judgments-pipeline`
        to the `staging` branch

    3. Merge any Step Function updates in `da-transform-terraform-modules`
        to the `staging` branch

    4. Trigger a Terraform apply by merging the feature branch changes in
        `da-transform-terraform-environments` to the `staging` branch<sup>2</sup>

5. Promote changes to the `prod` environment:

    1. Update the Lambda Function version(s) in `prod-tfvars` in Parameter Store<sup>1</sup>

    2. Merge any Lambda Function updates in `da-transform-judgments-pipeline`
        to the `prod` branch

    3. Merge any Step Function updates in `da-transform-terraform-modules`
        to the `prod` branch

    4. Trigger a Terraform apply by merging the feature branch changes in
        `da-transform-terraform-environments` to the `prod` branch<sup>2</sup>

> <sup>1</sup> The change will not be applied until a Terraform update is triggered

> <sup>2</sup> An arbitrary change may be needed to trigger the pipeline, or
    it can be triggered manually in the console
