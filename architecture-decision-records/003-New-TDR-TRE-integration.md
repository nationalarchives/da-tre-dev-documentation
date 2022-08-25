# New TDR-TRE integration

* [New TDR-TRE integration](#new-tdr-tre-integration)
    * [Status](#status)
    * [Context](#context)
        * [TRE Event payload structure changes](#tre-event-payload-structure-changes)
            * [Message UUIDs](#message-uuids)
        * [SQS-backed SNS topic communication](#sqs-backed-sns-topic-communication)
        * [TDR to TRE new-bagit Event](#tdr-to-tre-new-bagit-event)
        * [TRE validate-bagit Process](#tre-validate-bagit-process)
            * [TRE bagit-validated Event](#tre-bagit-validated-event)
            * [TRE bagit-validation-error Event](#tre-bagit-validation-error-event)
    * [Decision](#decision)
    * [Consequences](#consequences)

## Status

Proposed

## Context

Current TDR-TRE integration is documented [here](../tdr-integration/README.md).

The following revisions to TRE have been proposed to standardise communication
between multiple event producers and consumers:

* TRE Event payload structure changes
* Use of SQS-backed SNS topic communication

### TRE Event payload structure changes

The following top-level JSON message format is proposed:

```json
{
  "version": "1.0.0",
  "timestamp": 1661340408275571000,
  "UUIDs": [
    {
      "example-producer-UUID": "c73e5ca7-cf87-442a-8248-e05f81361ae0"
    }
  ],
  "producer": {
    "name": "example-producer",
    "process": "example-process",
    "type": "",
    "environment": "",
    "event-name": "example-event"
  },
  "parameters": {
    "example-event": {
      
    }
  }
}
```

Prior schema used to perform top-level message validation (not `parameters`):

* [da-transform-judgments-pipeline/lib/tre_lib/tre_lib/schema.json](https://github.com/nationalarchives/da-transform-judgments-pipeline/blob/c5b05fa57b24f5b9f6aaed828f9ea0365e79a76d/lib/tre_lib/tre_lib/schema.json)

JSON message key descriptions:

| Key                    | Description                                                                           |
|------------------------|---------------------------------------------------------------------------------------|
| `version`              | Version of the message being sent                                                     |
| `timestamp`            | Creation time in nanoseconds UTC                                                      |
| `UUIDs`                | UUIDs for this message and all prior messages <sup>[1](#message-uuids)</sup>          |
| `producer`             | Dictionary specifying the event-name being sent and details about the producer        |
| `producer.name`        | The message producer (e.g. TRE, TDR, etc)                                             |
| `producer.process`     | The name of the process that creates the message (e.g. `validate-bagit`)              |
| `producer.type`        | The name of the consignment type being passed; `standard`, `judgment` or null         |
| `producer.environment` | The name of the environment that creates the message (e.g. `test`, `production`, etc) |
| `producer.event-name`  | The name of the event being propagated (e.g. `new-bagit`, `bagit-validated`, etc)     |
| `parameters`           | Data specific to `producer.event-name` in a sub-section with that name                |

#### Message UUIDs

The `UUIDs` key must contain a list of UUID objects such that:

1. Each UUID object must have a key that is the name of the producer (i.e.
   `producer.name`) concatenated with the string `-UUID` (e.g. `TRE-UUID`)
2. Each UUID object key must have a valid UUID value
3. The last UUID object in the list must be that of the current message

Example UUIDs section:

```json
{
  "UUIDs": [
    {"TDR-UUID": "c73e5ca7-cf87-442a-8248-e05f81361ae0"},
    {"TRE-UUID": "ec506d7f-f531-4e63-833e-841918105e41"},
    {"TRE-UUID": "3c1db304-090f-4b19-abfc-8618cc0e5875"}
  ]
}
```

> Multiple UUID values could be added by a Step Function if multiple Lambda 
  functions within the Step Function each append a UUID to the message chain

### SQS-backed SNS topic communication

The proposal is to move from this V1 approach:

```
       TDR        TDR       TRE        TRE          TRE        TRE Out /
      Retry     Process    tdr-in    Forwarder     Process     Editorial In
     +-----+     +---+     +-----+     +---+     +---------+     +-----+
+--> | SQS | --> | ? | --> | SQS | --> | λ | --> | Step Fn | --> | SNS |
|    +-----+     +---+     +-----+     +---+     +---------+     +-----+
|                                                     |
+-----------------------------------------------------+
```

To the following:

```
 tre-in                                                    tre-out
 +-----+                       1                           +-----+
 |     |       +-------- TRE Process ---------+         4  |     |
 |     | ----> | +-----+     +---+     +----+ | ---------> |     |
 |     |    2  | | SQS | --> | λ | --> | SF | |  3         |     |
 |     |   +-> | +-----+     +---+     +----+ | --+        |     |
 |     |   |   +------------------------------+   |        |     |
 |     |   |                                      |        |     |
 |     |   |         +----------------------------+        |     |
 | SNS |   |         |                                     | SNS |
 |     |   |         |  tre-internal        tre-forward    |     |      
 |     |   |         |     +-----+            +-----+      |     |
 |     |   |         +---> |     | ---------> | SQS | ---> |     |  
 |     |   |          4    | SNS |            +-----+      |     |
 |     |   |               |     | --+                     |     |
 |     |   |               +-----+   |                     |     |
 |     |   +-------------------------+                     |     |
 +-----+                                                   +-----+

1 : E.g. validate-bagit, dri-preingest-sip
2 : Process can subscribe to tre-in or tre-internal 
3 : Process can publish to tre-out or tre-internal
4 : SNS publish call must add message's producer fields as SNS MessageAttributes
```

### TDR to TRE new-bagit Event

The current proposal is for TDR to send a `new-bagit` event to the TRE system.

The TRE [`validate-bagit`](#tre-validate-bagit-process) process will listen for 
these events and process them.

The TDR system is expected to set the following properties in the event payload
it sends:

In the `producer` section:

| Message Field         | Value                    |
|-----------------------|--------------------------|
| `producer.name`       | `TDR`                    |
| `producer.type`       | `standard` or `judgment` |
| `producer.process`    | ?                        |
| `producer.event-name` | `new-bagit`              |

In the `parameters` section:

| Message Field                                    | Value                                      |
|--------------------------------------------------|--------------------------------------------|
| `parameters.new-bagit.resource.value`            | BagIt archive file pre-signed URL          |
| `parameters.new-bagit.resource-validation.value` | BagIt archive checksum file pre-signed URL |
| `parameters.new-bagit.reference`                 | Consignment reference                      |

Message and schema examples:

* [da-transform-schemas/json-examples/tdr-to-tre-example.json](https://github.com/nationalarchives/da-transform-schemas/blob/main/json-examples/tdr-to-tre-example.json)
* [da-transform-schemas/json-schemas/tdr-to-tre-schema.json](https://github.com/nationalarchives/da-transform-schemas/blob/main/json-schemas/tdr-to-tre-schema.json)

### TRE validate-bagit Process

The `validate-bagit` process will process `new-bagit` events and either:

* Succeed and output a `bagit-validated` event
* Fail to validate the BagIt and output a `bagit-validation-failed` event
* Fail with a Step Function error; in this case no TRE event will be outputted 

On receipt of an event the process will:

* Create a working directory in S3 that has the input consignment reference as
  its name
* Create a directory in the above working directory that has the message's
  UUID as its name; enter a fail state if a child directory already exists with
  the UUID name
    > Other approaches such as sending failures, or acting in an idempotent way could be considered here
* The working folder will be used to store, unpack and verify the BagIt file

#### TRE bagit-validated Event

On successful validation of an input BagIt a `bagit-validated` event will be
published to the `tre-internal` SNS topic with the following data fields set:

In the `producer` section:

| Message Field         | Value                              |
|-----------------------|------------------------------------|
| `producer.name`       | `TRE`                              |
| `producer.type`       | From input message `producer.type` |
| `producer.process`    | `validate-bagit`                   |
| `producer.event-name` | `bagit-validated`                  |

In the `parameters` section:

| Message Field                                | Value                                                     |
|----------------------------------------------|-----------------------------------------------------------|
| `parameters.bagit-validated.reference`       | From input message `parameters.new-bagit.reference`       |
| `parameters.bagit-validated.s3-bucket`       | The TRE S3 bucket used to extract and verify the BagIt    |
| `parameters.bagit-validated.s3-bagit-name`   | The TRE S3 path of the saved input BagIt file             |
| `parameters.bagit-validated.s3-object-root`  | The TRE S3 folder where the input BagIt file is extracted |
| `parameters.bagit-validated.validated-files` | A dictionary with lists of validated files                |

Message and schema examples:

* [da-transform-schemas/json-examples/validate-bagit-bagit-validated.json](https://github.com/nationalarchives/da-transform-schemas/blob/main/json-examples/validate-bagit-bagit-validated.json)
* [da-transform-schemas/json-schemas/validate-bagit-bagit-validated-schema.json](https://github.com/nationalarchives/da-transform-schemas/blob/main/json-schemas/validate-bagit-bagit-validated-schema.json)

#### TRE bagit-validation-error Event

If an input BagIt fails validation, a `bagit-validation-error` event will be
published to the `tre-internal` SNS topic with the following data fields set:

In the `producer` section:

| Message Field         | Value                              |
|-----------------------|------------------------------------|
| `producer.name`       | `TRE`                              |
| `producer.type`       | From input message `producer.type` |
| `producer.process`    | `validate-bagit`                   |
| `producer.event-name` | `bagit-validation-error`           |

In the `parameters` section:

| Message Field                                 | Value                                               |
|-----------------------------------------------|-----------------------------------------------------|
| `parameters.bagit-validation-error.reference` | From input message `parameters.new-bagit.reference` |
| `parameters.bagit-validation-error.errors`    | A list of errors from the `validate-bagit` process  |

Message and schema examples:

* [da-transform-schemas/json-examples/validate-bagit-bagit-validation-error.json](https://github.com/nationalarchives/da-transform-schemas/blob/main/json-examples/validate-bagit-bagit-validation-error.json)
* [da-transform-schemas/json-schemas/validate-bagit-bagit-validation-error-schema.json](https://github.com/nationalarchives/da-transform-schemas/blob/main/json-schemas/validate-bagit-bagit-validation-error-schema.json)

## Decision

What is the change that we're proposing and/or doing?

## Consequences

What becomes easier or more difficult to do because of this change
