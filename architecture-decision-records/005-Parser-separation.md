# Parser separation proposal

Date: 2022-10-28

## Status

Accepted.

## Context

In TRE V1 the judgment parser component ran inside a single parent process.

In TRE V2 the judgment parser component will run as a separate process that
can be run by TRE processes and systems external to TRE.

> Note: event naming is subject to change;
  `bagit-validated` may change to `validated-bag-available`;
  `run-judgment-parser` may change to `judgment-available`.

## Design

The following high-level design options have been considered for the TRE V2
"Run Judgment Parser" process:

* Option 1: Accept only one event type as input
* Option 2: Accept multiple event types as input

### Option 1: Single Input Event

With this option, the "Run Judgment Parser" process would only accept a single
input event (e.g. `judgment-available`).

The benefit of this option is that the process would not need to be adapted
to handle responding to new events (that may have different parameters).

A downside to this approach is that an additional process will be needed to
identify any other events that should trigger the "Run Judgment Parser"
process. This process would be responsible for generating a valid
`judgment-available` event to subsequently trigger the "Run Judgment Parser"
process. An example of such an event is a `bagit-validated` event for a
`judgment`.

The additional process can be thought of as a routing "funnel" that would only
emit `judgment-available` events.

The following diagram shows some example TRE event flows for this scenario;
see note above regarding possible name changes:

![pic1](images/TREv2-judgment-parser.png)

### Option 2: Multiple Input Events

With this option, the "Run Judgment Parser" process would accept multiple
different events as input (for example, perhaps `judgment-available` and
`bagit-validated`).

The benefit of this approach is that a "routing" process would not be needed.

Downsides are that the "Run Judgment Parser" process may need:

* Event-specific logic (e.g. if events have different parameters)
* Updates to support responding to additional (or new) events

## Decision

It was agreed in design review (2022-10-27) to use option 1 for single input
events. This was deemed to be a more decoupled solution that would be easier
to extend.

## Consequences

Selecting option 1 means an additional routing process must be implemented.
