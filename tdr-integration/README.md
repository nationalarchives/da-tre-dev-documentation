# TDR integration

## Table of Contents
- [Introduction](#introduction)
- [Message exchange method and format](#message-exchange-method-and-format)
- [Credentials exchange and bucket permissions](#credentials-exchange-and-bucket-permissions)

## Introduction

Transfer Digital Record (TDR) will produce a [BagIt package](https://datatracker.ietf.org/doc/html/rfc8493) containing the judgment and technical metadata extracted as part of the existing TDR processes. TDR will notify Transformation Engine (TE) that the package is ready, and will provide one-time credentials for retrieval.

## Message exchange method and format

TE will exchange messages with TDR using an AWS SQS queue as per the [MVP Beta technical design](./../beta-mvp-architecture/README.md). The message will have the following format:

```json
{
  "consignment-reference" : "TDR-2021-GW3",
  "s3-bagit-url" : "https://presignedurldemo.s3.eu-west-2.amazonaws.com/image.png",
  "s3-sha-url" : "https://presignedurldemo.s3.eu-west-2.amazonaws.com/image.png",
  "consignment-type" : "Court Judgment",
  "number-of-retries" : 0
}

```

## Credentials exchange and bucket permissions

In the AWS Account where the TDR system is provisioned there will be AWS IAM Roles defined, based on prod and non-prod environments. In the same way, in the AWS Account where the TE is provisioned there will be AWS IAM Roles defined, based on prod and non-prod environaments.

TDR IAM Roles will have permissions to write a message to the AWS SQS queue in the AWS account where the TE is provisioned, and TE IAM Roles will have permissions to write a message to the AWS SQS queue where the TDR is provisioned.

The message will contain [S3 presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html) to retrieve the objects from the TDR S3 bucket.
