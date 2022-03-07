# Editorial System integration

## Table of Contents
- [Introduction](#introduction)
- [Message exchange method and format](#message-exchange-method-and-format)
- [Credentials exchange and bucket permissions](#credentials-exchange-and-bucket-permissions)
- [Retry mechanism](#retry-mechanism)

## Introduction

![pic1](../beta-mvp-architecture/diagrams/aws-step-function-workflow-for-te.png)

The TE will produce the following outputs for the Editorial system:
1. the data payload (the judgment itself)
2. the XML outputs of the parser 
3. any processing errors so the editorial team can make an assessment on whether the publishing can go ahead
4. a metadata file (in a JSON format) which contains the TE version and the text Parser version, as shown below
```json
{
  "te-version" : "1.0.0",
  "text-parser-version" : "1.0.1",
  "uploader-email" : "sample@test.com"
}
```

## Message exchange method and format

TE will exchange messages with Editorial system using an AWS SQS queue as per the [MVP Beta technical design](./../beta-mvp-architecture/README.md). The message will have the following format:

```json
{
  "consignment-reference": "TDR-2022-C67H",
  "s3-folder-url": "https://te-editorial-out-int.s3.eu-west-2.amazonaws.com/TDR-2022-C67H.tar.gz?X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJf%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCWV1LXdlc3QtMiJIMEYCIQDLTmaISu2r83kDSVlR%2F1uF1CgUv5rzy4iCG8jsG9%2F5wgIhAIyi2R%2F7XSdm6h7KLPBB6J0RVUCFO%2FWTgBL%2F1vtzKA3YKqQCCPD%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEQAhoMMjI5NTU0Nzc4Njc1Igye71PDmIudFYcsh5cq%2BAET%2F7sg5ecibhtmTRT7wpgPySuAet%2FABHE%2B49m53K7v4%2F3NTtTf9grvosoN9iO0DbvoaKaMccLJRlvCeLJ0Alsmh2NQf6w91t75AYKiMQ6oLIXmwc05Ewj2kND3L8uwBsjRoFNReNrWXFW1D7jiqiQpAw6BA05UW9K8%2Fwip55WrYyzBt84btm0Zy2ryVwseFEaOsEHHjglt%2Bl83r1GkGt7iLcJ7ZpvSAp6WjI3zRxqzyXc%2B%2Bs0IsJukZ9P3NQZIbeUADegqDhOq6BHoLTYryajRrSUX3WiGFf%2B7%2FVI55muASwtUYCkR20wQutty2xKMXGVwlZSgpw%2BkiTDsuPOQBjqZAQafrMvrVG5FO5mPvY8rwVfag9z%2FhiOP7juMJ7GEwaNoMouZJvbslIGk6mfOWxfZ1qE99oRlKt9YuF1u53bPynC17N1vfIr9kW2bEsDLxndz35ZRR2yEjwhgwulbZdykVxjc%2BM%2BiDiGrtj53%2BlAOy8xSFZl1AMkBuAtNjAsTqGNyHzLQSJfeb83%2BoOIPe4O4%2B7wM7hStr28rAQ%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220228T143015Z&X-Amz-SignedHeaders=host&X-Amz-Expires=60&X-Amz-Credential=ASIATK4UH6IZTWHYAHCS%2F20220228%2Feu-west-2%2Fs3%2Faws4_request&X-Amz-Signature=c39106d6f8b2e23ab4c896825bdc4489f4fb4bb41d93f4b9ec70e52aa0ad8399",
  "consignment-type": "judgment",
  "number-of-retries": 0
}
```

## Credentials exchange and bucket permissions

In the AWS Account where the Editorial system is provisioned there will be AWS IAM Roles defined, based on prod and non-prod environments.

Editorial IAM Roles will have permissions to subscribe to the AWS SQS queue in the AWS account where the TE is provisioned.

The message will contain [S3 presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html) to retrieve the objects from the TDR S3 bucket.

## Retry mechanism

The retry mechanism is implemented using an additional AWS SQS queue in the AWS where the TE is deployed, as shown in the diagram above:

1. the editorial system should provide TE with an AWS IAM Role in order to be able to write messages to the queue
2. the structure of the message is the same structure defined above, with the increment of the field "number-of-retries" an empty field for "s3-folder-url" 
```json
  {
    "consignment-reference": "TDR-2022-C67H",
    "s3-folder-url": "",
    "consignment-type": "judgment",
    "number-of-retries": 1
  }
```
3. when a new message is sent, the TE will trigger again the text parser stage 
4. once the text parser stage is complete, TE will notify again the editorial system about the new output from the text parser
5. the max number of ritries is set to 3
6. after 3 attempts TE will go to a state error
7. for each retry, the editorial system will increment the field "number-of-retries"
