# Simulating A TDR Bag Available Event In TRE v2

This run-book describes how to manually trigger TRE v2 processing by sending a
`bagit-available` event to TRE's input SNS topic (simulating a TDR input).

* [Clone Repository da-transform-judgments-pipeline](#clone-repository-da-transform-judgments-pipeline)
* [Setup Environment Variables](#setup-environment-variables)
* [Input Verification](#input-verification)
* [Execution](#execution)
* [Output Verification](#output-verification)
* [Cleanup](#cleanup)

## Clone Repository da-transform-judgments-pipeline

* Clone da-transform-judgments-pipeline

  ```bash
  git clone git@github.com:nationalarchives/da-transform-judgments-pipeline.git
  ```

## Setup Environment Variables

* Select a test consignment; this example will use `TDR-2022-D6WD` (from
  [da-tre-sample-data](https://github.com/nationalarchives/da-tre-sample-data/tree/main/test/resources/small-normal-batch/bagit/v1-2)):

  ```bash
  CONSIGNMENT_REF='TDR-2022-D6WD'
  ```

* Identify your AWS profile names for the AWS management and non-prod accounts:

  ```bash
  AWS_PROFILE_MANAGEMENT="${your_aws_profile_name_for_management_account}"
  AWS_PROFILE_NON_PROD="${your_aws_profile_name_for_non_prod_account}"
  ```

* Identify the s3 bucket that holds the source consignment `tar.gz` and
  `tar.gz.sha256` files:

  ```bash
  TEST_DATA_BUCKET="${s3_bucket_name_holding_test_consignment_files}"
  ```

* Identify the s3 buckets used by the bag validation and DRI pre-ingest SIP
  generation processes:

  ```bash
  VB_BUCKET="${s3_bucket_name_for_bag_validation_process}"
  DPSG_BUCKET="${s3_bucket_name_for_dri_preingest_sip_generation_process}"
  ```

* Identify the ARN of the TRE input SNS topic:

  Determine the topic name (i.e. `${env}-tre-in`):

  ```bash
  TRE_IN_TOPIC_NAME=${sns_input_topic_name}
  ```

  Determine the topic's ARN; e.g.:

  ```bash
  AWS_CLI_QUERY='Topics[?ends_with(TopicArn, `:'"${TRE_IN_TOPIC_NAME:?}"'`) == `true`].TopicArn | [0]'

  TRE_IN_TOPIC_ARN="$(
    aws --profile "${AWS_PROFILE_NON_PROD:?}" \
      sns list-topics \
        --query "${AWS_CLI_QUERY:?}" \
        --output text
  )"
  ```

## Input Verification

* Confirm the test consignment has `tar.gz` and `tar.gz.sha256` files present:

  ```bash
  aws --profile "${AWS_PROFILE_MANAGEMENT:?}" \
    s3 ls "s3://${TEST_DATA_BUCKET}/consignments/standard/${CONSIGNMENT_REF:?}"
  ```

## Execution

* Navigate to the `tdr_sns_in_publish` directory:

  ```bash
  cd da-transform-judgments-pipeline/testing/tdr_sns_in_publish
  ```

* Execute the `run.sh` script with the required arguments:

  ```bash
  ./run.sh "${TRE_IN_TOPIC_ARN:?}" \
    "${TEST_DATA_BUCKET:?}" \
    consignments/standard/"${CONSIGNMENT_REF:?}".tar.gz \
    consignments/standard/"${CONSIGNMENT_REF:?}".tar.gz.sha256 \
    "${CONSIGNMENT_REF:?}" \
    standard \
    "${AWS_PROFILE_MANAGEMENT:?}" \
    "${AWS_PROFILE_NON_PROD}"
  ```

## Output Verification

To view output data created for the consignment reference run the following:

  ```bash
  aws --profile "${AWS_PROFILE_NON_PROD:?}" \
    s3 ls \
      --recursive \
      "s3://${VB_BUCKET:?}/consignments/standard/${CONSIGNMENT_REF:?}/"
  ```

  ```bash
  aws --profile "${AWS_PROFILE_NON_PROD:?}" \
    s3 ls \
      --recursive \
      "s3://${DPSG_BUCKET:?}/consignments/standard/${CONSIGNMENT_REF:?}/"
  ```

> The UUID in the s3 path can be verified against the Step Function execution

## Cleanup

* You may wish to remove all output data from prior runs; to do this:

  ```bash
  aws --profile "${AWS_PROFILE_NON_PROD:?}" \
    s3 rm \
      --recursive \
      "s3://${VB_BUCKET:?}/consignments/standard/${CONSIGNMENT_REF:?}/"
  ```

  ```bash
  aws --profile "${AWS_PROFILE_NON_PROD:?}" \
    s3 rm \
      --recursive \
      "s3://${DPSG_BUCKET:?}/consignments/standard/${CONSIGNMENT_REF:?}/"
  ```
