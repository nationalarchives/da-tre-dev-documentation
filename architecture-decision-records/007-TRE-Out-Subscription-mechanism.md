# TRE-Out-Subscription-mechanism

Date: 15-09-2022

## Status

Accepted

## Context

Aggreing on how external teams can subscribe to TRE-Out SNS Topic.

### Option 1

Team that wishes to subscribe (TDR, DRI etc) creates a role which has `sns:Subscribe` permission, given by team that owns TRE-Out SNS Topic (TRE), to create subscription.

* Pros
    * Give other teams (TDR, DRI etc) ability to subscribe without needing any asistance from TRE team.
    * Easier to implement for both teams.
* Cons
    * Anyone who assumes the Role created by the other team (TDR, DRI etc) can create any subscripton. (Goes against 'The principle of least privilege')

### Option 2 - SQS Subscription Method

TRE-Out SNS Topic Owner (TRE) creates subscription using the endpoint given by the other team (TDR, DRI etc)

Other teams (TDR, DRI etc) provide TRE a SQS ARN as an endpoint which can be used to create subscription manually using AWS Consle. After that a message is sent to the SQS Queue which includes 'SubscribeURL' that needs to be visted to confirm the subscription.

> **Warning**
>
> Setup of cross-account subscriptions from SNS topics to SQS queues requires Terraform to have access to BOTH accounts. Click [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/sns_topic_subscription) for details.

* Pros
    * Adhering to the principle of least privilege
* Cons
    * Manual process
    * Requires actions from both teams

### Option 3 - Lambda Subscription Method

TRE creates a Role which has `lambda:AddPermission` on endpoint Lambda given by the other teams (TDR, DRI etc) to create TRE-Out SNS Subscription which will create a Trigger for the endpoint Lambda.

* Pros
    * No action required from the other teams (TDR, DRI etc)
    * Easier to implement for both teams.
* Cons
    *  Anyone who assumes the Role created by TRE can create triggers for lambda that is owned by the other teams (TDR, DRI etc). (Goes against 'The principle of least privilege')

# Decision

Option-2 with the SQS subscription method is the preferred option.

# Consequences

- The consumers which want to subscribe to TRE-OUT topic have to send a request to TRE team with SQS ARN and endpoint
- TRE team has to create the subscription manually using the AWS console

# New decision

The owner of the topic will add a policy that allows permitted accounts to publish to the topic and/or a permitted account to subcribe a sepcific endpoint to the topic
* The external account will need to provide the topic owner their account id and subscription endpoint.
* The policies will be added with terraform
