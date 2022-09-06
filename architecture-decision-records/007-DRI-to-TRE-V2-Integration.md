# DRI to TRE V2 Integration

Date: 05-09-2022

## Status

In progress

## Context

In order to facilitate integration with DRI, message payloads will need to contain headers that can be ingested by DRI. Further context as to message structure can be found 
on the [TDR -TRE integration page](https://github.com/nationalarchives/da-transform-dev-documentation/blob/master/architecture-decision-records/003-New-TDR-TRE-integration.md#tdr-to-tre-new-bagit-event)

## Decision

The payload must contain the event-name "dri-preingest-sip-available" which is identified under the producer header. 

| Message Field         | Value                                                                                               |
|-----------------------|-----------------------------------------------------------------------------------------------------|
| `producer.event-name` | The name of the event sending this payload, for now this will always be dri-preingest-sip-available |

This event name is also the header contained under the parameters header

Most message attributes are inherited from the parent schema ([Tre-Event Schema](https://github.com/nationalarchives/da-transform-schemas/blob/main/tre_schemas/tre-event.json)
/ [Tre-Event Example]()) However when TRE interacts with DRi it should contain the payload should contain the following attributes under their respective headers

| Message Field                                              | Value                                                                                |
|------------------------------------------------------------|--------------------------------------------------------------------------------------|
| `parameters.dri-preingest-sip-available.reference`         | reference for the attached object                                                    |
| `parameters.dri-preingest-sip-available.s3-folder-name`    | url to directory containing bagit object in AWS s3                                   |
| `parameters.dri-preingest-sip-available.s3-sha256-url`     | url to the bagit object itself in AWS s3                                             |
| `parameters.dri-preingest-sip-available.number-of-retries` | The amount of times this message has been resent to DRI from TRE                     |
| `parameters.dri-preingest-sip-available.file-types`        | the format of the file or archive i.e. PDF, JPEG, ZIP, TAR  (this field is optional) |

## Consequences

Message payloads between DRI and TRE will have to conform to the headings outlined in this article.
