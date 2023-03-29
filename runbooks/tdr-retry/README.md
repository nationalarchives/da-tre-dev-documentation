# Manually Triggering A TDR Retry Request

```diff 
- This relates to TRE V1
```

A TDR retry can be triggered using the following manual steps:

1. Find the Consignment's most recent execution in the `Step Functions`
    section of the AWS console<sup>1</sup>

    > Use the `Search for executions` filter to only show records for a
        particular consignment reference

2. Click the latest record to view its execution graph
3. Select the `Bagit checksum validation` step
4. Copy the step's `Input` JSON (ensure the `Input & Output` tab is selected)

    > This input JSON is the payload received from TDR

5. Navigate to the TDR input SQS queue (`"${env}-tre-tdr-in"`)
6. Click the `Send and receive messages` button
7. Past the JSON copied in step 4 above into the `Message body` field
8. Remove the values for JSON keys `s3-bagit-url` and `s3-sha-url`

    > The message will look something like this (number-of-retries field may
        be non zero):

    ```json
    {
      "consignment-reference": "TDR-....-...",
      "s3-bagit-url": "",
      "s3-sha-url": "",
      "consignment-type": "judgment",
      "number-of-retries": 0
    }
    ```

9. Click the `send message` button

Re-submitting the last known TDR message to TRE causes the TRE process to fail
(because the `number-of-retries` field will already have been processed); the
TRE process then sends a retry request message to TDR.

> <sup>1</sup> An alternative method to identify a consignment's latest TDR
    `number-of-retries` value is to locate it in the S3 path under one of the
    TRE process' S3 buckets. In S3 bucket `"${env}-tre-temp"` this value can
    be found under `"consignments/judgment/${consignment_reference}/"`. In S3
    bucket `"${env}-tre-editorial-judgment-out"` this value can be found under
    `"parsed/judgment/${consignment_reference}/"`.
