# Beta MVP architecture

## Table of Contents
- [Introduction](#introduction)
- [Beta MVP AWS Architecture diagram](#beta-mvp-aws-architecture-diagram)
    - [Roles used in the solution](#roles-used-in-the-solution)
    - [AWS Services used in the solution](#aws-services-used-in-the-solution)
    - [Integration with Transfer Digital Records (TDR)](#integration-with-transfer-digital-records)
    - [Integration with the Judgments parser](#integration-with-the-judgments-parser)
- [AWS Quotas for the services](#aws-quotas-for-the-services)
- [AWS Accounts](#aws-accounts)
- [Terraform](#terraform)

### Introduction

TO DO

### Beta MVP AWS Architecture diagram

The following architecture diagram simplifies the proposed solution for the Beta MVP Transformation pipeline.

![pic1](./diagrams/da-transform-beta-mvp-aws-architecture-diagram.png)

#### Roles used in the solution

* Developers
* DevOps
* Administrators
* TNA users
* Workflow editors

#### AWS Services used in the solution

* [Amazon Step Functions](https://docs.aws.amazon.com/step-functions/latest/dg/getting-started.html), 
* [Amazon Elastic Kubernetes Service (EKS)](https://aws.amazon.com/eks), gives you the flexibility to start, run, and scale Kubernetes applications in the AWS cloud.
* [AWS Fargate](https://aws.amazon.com/fargate) is a serverless compute engine for containers that works with both [Amazon Elastic Container Service (ECS)](https://aws.amazon.com/ecs/) and [EKS](https://aws.amazon.com/eks/). Fargate makes it easy for you to focus on building your applications. Fargate removes the need to provision and manage servers, lets you specify and pay for resources per application, and improves security through application isolation by design.
* [Amazon Elastic Container Registry (ECR)](https://aws.amazon.com/ecr/) is a fully managed container registry that makes it easy to store, manage, share, and deploy your container images and artifacts anywhere.
* [AWS Identity and access management for Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/security-iam.html), IAM administrators control who can be authenticated (signed in) and authorized (have permissions) to use Amazon EKS resources. [OpenID Connect (OIDC) Identity Provider (IDP)](https://aws.amazon.com/blogs/containers/introducing-oidc-identity-provider-authentication-amazon-eks/) feature allows customers to integrate an OIDC identity provider with the Amazon EKS cluster running Kubernetes version 1.16 or later. With this feature, you can manage user access to your cluster by leveraging existing identity management life cycle through your OIDC identity provider. 
    * Additionally, you can enhance this solution with the combination of public OIDC endpoint and IRSA. Administrators and Developers can put the IAM role to a specific pod or restrict to a single IP range of the pod to provide fine grained access.
* [Amazon Simple Email Service (SES)](https://aws.amazon.com/ses/), 
* [Amazon Simple Queue Service (SQS)](https://aws.amazon.com/sqs/),
* [Amazon Simple Notification Service (SNS)](https://aws.amazon.com/sns/),
* [Amazon Serverless Computing - AWS Lambda](https://aws.amazon.com/lambda/),
* [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/),
* [Amazon CloudTrail](https://aws.amazon.com/cloudtrail/),
* [Amazon GuardDuty](https://aws.amazon.com/guardduty/),
* [Amazon X-Ray](https://aws.amazon.com/xray/),

#### Integration with Transfer Digital Records

The Transfer Digital Records service will have two integration points with the Transformation Service:

1. the service should be able to send a message with metadata in the AWS SQS queue
2. the AWS S3 bucket where TDR stores the Bagit packages should be accessible by the AWS EKS cluster and the pods where the jobs will run

**Question**: What happens when there is an invalid checksum?

#### Integration with the Judgments parser

* This parser converts UK judgments from .docx format to XML. It is written in C# and requires .NET 5.0.
* The source code in GitHub ia available [here](https://github.com/mangiafico/tna-judgments)
* To execute the parser we have identified the following options:
    1. Execute the paser in a lambda function
    2. Wrap the parser in a docker container and use AWS EKS Fargate

#### AWS Step Functions

![pic2](./diagrams/aws-step-functions-workflow-console.png)


### AWS Quotas for the services

* [Amazon Step Functions Quotas](https://docs.aws.amazon.com/step-functions/latest/dg/limits-overview.html)


### AWS Accounts

TO DO

### Terraform

TO DO