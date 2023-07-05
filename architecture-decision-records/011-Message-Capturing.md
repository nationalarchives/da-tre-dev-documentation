# Message Capturing

## Status
Accepted

Note: At present this is only implemented for ```judgmentpackage.available.JudgmentPackageAvailable``` messages.  This can be easily implemented for all messages by changing the subscription filter to be wider and using a partition that will match all messages; or by creating a new firehose subscription with suitable partitioning.

## Context
Hitherto, it is not possible to see what messages are published by the ```tre-out``` SNS Topic.

A mechanism is needed so that message output from the ```tre-out``` SNS Topic can be inspected for the purposes of testing, debugging and possibly archiving.

## Decision
We will use Kinesis Firehose to subscribe to the SNS topic ```tre-out``` and dump a copy of each message to S3.

We will use portioning so that messages can be easily located in S3. e.g. a message is dumped in S3 with the following example path like prefix:
```<bucket name>/judgmentpackage.available.JudgmentPackageAvailable/FCL/FCL-TEST-123/<executionId uid>/<object name generated by firehose>```.

Messages that could not be partitioned will be dumped with a prefix (path) of ```errors/``` in the same bucket.

We will dump messages in its raw format.  i.e. The JSON message sent to be published by SNS is exactly what will dumped to S3.  The meta data that can be generated by SNS will not be used.  Therefore, the SNS subscription configuration ```Raw message delivery Enabled``` will be used.

## Other considered solutions
We will not use a lambda to subscribe to SNS and do something that way.  We prefer to use patterns and products that are available from AWS and can be configured without writing any code.

We will not use firehose to dump to Redshift.  The volume of messages is expected to be relatively low does not warrant the cost of standing up and running a Redshift instance.

## Consequences
It will now be possible to inspect messages that are produced from ```tre-out```.  We will be able to see what the customers sees.  This can be done easily by navigating through the AWS S3 console, or programmatically using the SDK etc.

This will enable systems testing - the original motivation for this work as messages could not be inspected before.

This may also double as an archiving solution assuming the buckets were never purged.  This could be further used as data source for AWS Athena, AWS Glue, Talend etc.

There will be small cost increase for S3 storage and firehose.  This is expected to be negligible.