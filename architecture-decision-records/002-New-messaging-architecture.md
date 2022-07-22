# New messaging architecture

Date: 21-07-2022

## Status

Proposed

## Context

The new [Enhanced message structure](./001-Enhanced-message-structure.md) requires a new architecture on messaging exchange between TRE and other systems.

> Phil, can you add more details in this section about the future plan?

## Options considered

See document [New messaging architecture](../technology-considerations/messaging-architecture/README.md) in the [technology considerations](../technology-considerations/) folder.

## Decision

The following is now proposed as the new messaging architecture:

![pic1](../technology-considerations/messaging-architecture/diagrams/tre-exchange-messages-option3.png)

TRE will have two SNS topics:

- **tre-in** where producers will send messages to TRE
- **tre-out** where consumers will receive messages from TRE

The diagram includes the Parser separation and two additional topics:

- **parser-in** where producers will send messages to the Parser Step Function
- **parser-out** where consumers will receive messages from the Parser Setp Function

For more details about the integrations between TRE and other systems have a look at the following pages:

- [New TDR-TRE integration](./003-New-TDR-TRE-integration.md)
- [New CaseLaw-TRE integration](./004-New-CaseLaw-TRE-integration.md)
- [Parser separation](./005-Parser-separation.md)

## Consequences

The event bus is implemented using SNS topics and there is no logic for routing the messages to a specific consumer: the consumers are responsable of filtering the messages, accepting or rejecting them based on their implementation. With this approach there is not any dependency between the event bus and the consumers.

With this architecture any integration with TRE is much more simpler, the new service has to subscribe to the two topics **tre-in** and **tre-out**. 

What becomes easier or more difficult to do because of this change?