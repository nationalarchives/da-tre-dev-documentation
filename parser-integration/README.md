# Parser integration

## Table of Contents
- [Introduction](#introduction)
- [Architecture Diagram](#architecture-diagram)
    - [Integration](#integration)
    - [Data flow](#data-flow)
- [References](#references)

## Introduction

The text parser is a tool being actively developed by a third party that extracts data properties from the text of court judgment documents. It utilises Microsoft’s Office Open XML SDK (Software Development Kit) and is written with the .NET technology stack. 

## Architecture diagram

The following architecture diagram simplifies the proposed solution for Parser integration.

![pic1](./diagrams/tna-parser-integration.png)

### Integration

The source code of the text parser is stored in a GitHub [repository](https://github.com/mangiafico/tna-judgments). As shown in the above diagram:


1. Amazon API Gateway receives Git webhook requests and forward them to AWS Lambda.
2. An AWS Lambda function processes Git webhook requests from API Gateway and invoke an AWS CodeBuild project.
3. An AWS CodeBuild project connetcs to Git service, then retrieves and uploads the latest version of the Git repository to Amazon CodeCommit.
4. An AWS Key Management Service (AWS KMS) key to encrypt/decrypt the SSH (Secure Shell) keys used by AWS CodeBuild to connect to Git repository using SSH.

### Data flow

1. TDR will produce a [BagIt package](https://datatracker.ietf.org/doc/html/rfc8493) containing the judgment and technical metadata extracted as part of the existing TDR processes
2. TDR will notify Transformation Engine that the package is ready, and will provide one-time credentials for retrieval 
3. When a new message is sent to the SQS, it triggers a Lambda Function which invokes AWS Step Function
4. AWS Step Function will start the workflow:
    1. A Lambda Function is triggered which acquires the package and validates checksum for both, BagIt package and .docx file extracted
    2. If the checksum validation fails, the workflow will retry N-times 
    4. If the checksum validation passes, another Lambdda Function will run the text parser against the judgment, which will extract data properties from the text and compile to XML
    5. The outputs of the text parser will be saved to the Parser OUT S3 bucket

## References

- https://aws.amazon.com/quickstart/architecture/git-to-s3-using-webhooks/