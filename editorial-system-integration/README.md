# Editorial System integration

```diff
- This relates to TRE V1
```

## Table of Contents
- [Introduction](#introduction)
- [Message exchange method and format](#message-exchange-method-and-format)
- [Credentials exchange and bucket permissions](#credentials-exchange-and-bucket-permissions)
- [Retry mechanism](#retry-mechanism)

## Introduction

![pic1](../beta-mvp-architecture/diagrams/aws-step-function-workflow-for-te.png)

The integration between the Transformation Engine (TRE) and the Editorial system is implemented using an TRE AWS SNS and a Lambda function in the AWS account where the Editorial system is provisioned which subscribes to the TRE SNS topic. TRE will notify Editorial system when the outputs are ready, and will provide one-time credentials for retrieval. The diagram above shows the integration.

The TRE will produce the following outputs for the Editorial system:
1. the data payload (the judgment itself)
2. the XML outputs of the parser
3. any processing errors so the editorial team can make an assessment on whether the publishing can go ahead
4. a metadata file (in a JSON format) which contains the TRE version and the text Parser version, as shown below
```json
{
  "producer": {
    "name": "TRE",
    "process": "transform",
    "type": "judgment"
  },
  "parameters": {
    "TRE": {
      "reference": "TRE-TDR-2022-DGQ",
      "payload": {
        "filename": "Cook UK v Boston Scientific - Approved Judgment - 08.03.22 (004).docx",
        "xml": "TDR-2022-DGQ.xml",
        "metadata": "TRE-TDR-2022-DGQ-metadata.json",
        "images": [
          "image1.png"
        ],
        "log": "parser.log"
      }
    },
    "PARSER": {
      "uri": "https://caselaw.nationalarchives.gov.uk/id/ewhc/pat/2022/504",
      "court": "EWHC-Chancery-Patents",
      "cite": "[2022] EWHC 504 (Pat)",
      "date": "2022-03-08",
      "name": null,
      "attachments": [ ],
      "error-messages": [ ]
    },
    "TDR": {
      "Consignment-Type": "judgment",
      "Bag-Creator": "TDRExportv0.0.87",
      "Consignment-Start-Datetime": "2022-06-21T17:10:19Z",
      "Consignment-Series": "",
      "Source-Organization": "HM Courts and Tribunals Service",
      "Contact-Name": "Pauline Drewett",
      "Internal-Sender-Identifier": "TDR-2022-DGQ",
      "Consignment-Completed-Datetime": "2022-06-21T17:12:06Z",
      "Consignment-Export-Datetime": "2022-06-21T17:12:57Z",
      "Contact-Email": "pauline.drewett@justice.gov.uk",
      "Payload-Oxum": "43102.1",
      "Bagging-Date": "2022-06-21"
    }
  }
}
```

## Message exchange method and format

TRE will exchange messages with Editorial system using an AWS SNS as per the [MVP Beta technical design](./../beta-mvp-architecture/README.md). The message will have the following format:

```json
{
  "consignment-reference": "TDR-2022-C67H",
  "s3-folder-url": "https://te-editorial-out-int.s3.eu-west-2.amazonaws.com/TDR-2022-C67H.tar.gz?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCWV1LXdlc3QtMiJIMEYCIQDLTmaISu2r83kDSVlR%2F1uF1CgUv5rzy4iCG8jsG9%2F5wgIhAIyi2R%2F7XSdm6h7KLPBB6J0RVUCFO%2FWTgBL%2F1vtzKA3YKqQCCPD%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQAhoMMjI5NTU0Nzc4Njc1Igye71PDmIudFYcsh5cq%2BAET%2F7sg5ecibhtmTRT7wpgPySuAet%2FABHE%2B49m53K7v4%2F3NTtTf9grvosoN9iO0DbvoaKaMccLJRlvCeLJ0Alsmh2NQf6w91t75AYKiMQ6oLIXmwc05Ewj2kND3L8uwBsjRoFNReNrWXFW1D7jiqiQpAw6BA05UW9K8%2Fwip55WrYyzBt84btm0Zy2ryVwseFEaOsEHHjglt%2Bl83r1GkGt7iLcJ7ZpvSAp6WjI3zRxqzyXc%2B%2Bs0IsJukZ9P3NQZIbeUADegqDhOq6BHoLTYryajRrSUX3WiGFf%2B7%2FVI55muASwtUYCkR20wQutty2xKMXGVwlZSgpw%2BkiTDsuPOQBjqZAQafrMvrVG5FO5mPvY8rwVfag9z%2FhiOP7juMJ7GEwaNoMouZJvbslIGk6mfOWxfZ1qE99oRlKt9YuF1u53bPynC17N1vfIr9kW2bEsDLxndz35ZRR2yEjwhgwulbZdykVxjc%2BM%2BiDiGrtj53%2BlAOy8xSFZl1AMkBuAtNjAsTqGNyHzLQSJfeb83%2BoOIPe4O4%2B7wM7hStr28rAQ%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220228T143015Z&X-Amz-SignedHeaders=host&X-Amz-Expires=60&X-Amz-Credential=ASIATK4UH6IZTWHYAHCS%2F20220228%2Feu-west-2%2Fs3%2Faws4_request&X-Amz-Signature=c39106d6f8b2e23ab4c896825bdc4489f4fb4bb41d93f4b9ec70e52aa0ad8399",
  "consignment-type": "judgment",
  "number-of-retries": 0
}
```

## Credentials exchange and bucket permissions

In the AWS Account where the Editorial system is provisioned there will be AWS IAM Roles defined, based on prod and non-prod environments. In the same way, in the AWS Account where the TRE is provisioned there will be AWS IAM Roles defined, based on prod and non-prod environaments.

TRE IAM Roles will have permissions to write a message to the AWS SQS queue in the AWS account where the Editorial system is provisioned, and Editorial IAM Roles will have permissions to write a message to the AWS SQS queue where TRE is provisioned.

The message will contain [S3 presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html) to retrieve the objects from the TRE Court Judgment OUT S3 bucket.

## Retry mechanism

The retry mechanism is implemented using an additional AWS SQS queue in the AWS account where the TRE is deployed, as shown in the diagram above:

1. the editorial system should provide TRE with an AWS IAM Role in order to be able to write messages to the queue
2. the structure of the message is the same structure defined above, with the increment of the field "number-of-retries" and an empty field for "s3-folder-url"
```json
  {
    "consignment-reference": "TDR-2022-C67H",
    "s3-folder-url": "",
    "consignment-type": "judgment",
    "number-of-retries": 1
  }
```
3. when a new message is sent, the TRE will trigger again the text parser stage
4. once the text parser stage is complete, TRE will notify again the editorial system about the new output from the text parser
5. the max number of retries is set to 3
6. after 3 attempts TRE will go to a state error
7. for each retry, the editorial system will increment the field "number-of-retries"
