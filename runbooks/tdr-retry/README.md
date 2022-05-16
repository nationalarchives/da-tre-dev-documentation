# Manually Triggering A TDR Retry Request

A TDR retry can be triggered using the following manual steps:

1. Find the Consignment's most recent execution in the `Step Functions`
    section of the AWS console

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

> Re-submitting the last known TDR message to TRE causes the TRE process to
    fail (because the `number-of-retries` field will already have been
    processed); the TRE process then sends a retry request message to TDR.
