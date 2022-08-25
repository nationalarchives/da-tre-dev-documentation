# New TDR-TRE integration

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

JSON message key descriptions:

| Key                    | Description                                                                                 |
|------------------------|---------------------------------------------------------------------------------------------|
| `version`              | Version of the message being sent                                                           |
| `timestamp`            | Creation time in nanoseconds UTC                                                            |
| `UUIDs`                | UUIDs for this message and all prior messages, this message's UUID must be last in the list |
| `producer`             | Specifies the event-name being sent and details about the producer                          |
| `producer.name`        | The message producer (e.g. TRE, TDR, etc)                                                   |
| `producer.process`     | The name of the process that creates the message (e.g. `validate-bagit`)                    |
| `producer.type`        | The name of the consignment type being passed; `standard`, `judgment` or null               |
| `producer.environment` | The name of the environment that creates the message (e.g. `test`, `production`, etc)       |
| `producer.event-name`  | The name of the event being propagated (e.g. `new-bagit`, `bagit-validated`, etc)           |
| `parameters`           | Data specific to `producer.event-name` in a sub-section with that name                      |

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

## Decision

What is the change that we're proposing and/or doing?

## Consequences

What becomes easier or more difficult to do because of this change?