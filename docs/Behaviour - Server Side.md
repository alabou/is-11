# Behaviour: Server Side

## Active Constraints of Sender

The initial state of Active Constraints of a Sender MUST be an empty `constraint_sets` array. This state indicates a Sender is unconstrained.

Non-empty `constraint_sets` array MUST be treated as unordered unless at least one Constraint Set has `urn:x-nmos:cap:meta:preference` attribute.

Once a Sender accepts proposed Active Constraints, this Sender, the Flow and the Connection API `/transportfile` resource (if used) associated with this Sender and the Source associated with this Flow MUST satisfy the Active Constraints when the Sender is active.

### Request Methods

- `GET` request returns the last successfully applied Active Constraints.
- `PUT` request MUST be validated and if the Sender supports Parameter Constraints used in the proposed Active Constraints and is capable to adhere at least one of the Constraint Sets, then the associated Inputs may be reconfigured, which may eventually cause the update of the corresponding Flows, Sources, `/transportfile` and Sender. If successful, the operation MUST return the accepted Active Constraints, otherwise an error MUST be returned and nothing changed.
- `DELETE` request MUST clear Active Constraints. It turns the Sender to the initial state which excludes IS-11 out of the process.

## State of Sender

A Sender managed with IS-11 has five states:
- `Unconstrained` when Active Constraints of this Sender is an empty `constraint_sets` array.
- `Constrained` when this Sender, the Flow and the `/transportfile` (if used) associated with this Sender and the Source associated with this Flow satisfy the Active Constraints.
- `Active Constraints Violation` when this Sender, the Flow or the `/transportfile` (if used) associated with this Sender or the Source associated with this Flow does not satisfy the Active Constraints. When an inactive Sender is in this state, it MUST NOT allow [IS-05][IS-05] activations.
- `No Signal` when there is no signal from Inputs associated with this Sender.
- `Awaiting Signal` when Active Constraints was just PUT and the signal from Inputs associated with this Sender is not stable yet. This is a transitional state until one of the previous ones can be established.

## Preventing restrictions violation

At any time if State of the active Sender becomes `Active Constraints Violation`, the Sender MUST NOT transmit the Flow over the network. The Sender is REQUIRED to take the following actions:
- MUST set the [IS-05][IS-05] `/active`'s `master_enable` property to `false`.
- MUST update the [IS-04][IS-04] `subscription`'s `active` property to `false` (regardless of unicast or multicast).

If staged parameters update is requested for a Receiver with a `/transportfile` violating Receiver Capabilities, the Receiver is REQUIRED to refuse such request with `400 Bad Request`. If the Receiver is active, it also MUST stop receiving the Flow by taking the following actions:
- MUST set the [IS-05][IS-05] `/active`'s `master_enable` property to `false`.
- MUST update the [IS-04][IS-04] `subscription`'s `active` property to `false` (regardless of unicast or multicast).

If an active Receiver changes its Receiver Capabilities, then it MUST validate `/transportfile` against them and if the validation fails, it MUST stop receiving the Flow by taking the following actions:
- MUST set the [IS-05][IS-05] `/active`'s `master_enable` property to `false`.
- MUST update the [IS-04][IS-04] `subscription`'s `active` property to `false` (regardless of unicast or multicast).
## Inputs

### Properties

The `/properties` endpoint shows properties of the Input. In particular, it MAY contain information parsed from Base EDID (if set) and Effective EDID if the Input supports EDID and the Node is capable to parse EDID binaries.

### Base EDID

There is no Base EDID at the initial state. If Base EDID for an Input changes, then all Senders associated with this Input MUST update their versions (in registered mode this MUST update the registered resources).

### Effective EDID

Effective EDID is such combination of Base EDID and Active Constraints of all Senders associated with this Input that the baseband signal from the Input can be transmitted by the Senders without breaking Active Constraints.

If Base EDID is not set, Effective EDID is built on basis of a default Effective EDID defined for the Input by manufacturer.

The `/edid/effective` endpoint allows a client to download the Effective EDID if it exists. If the Effective EDID for the Input changes and it is associated with any Senders, then all of the Senders in question MUST update their versions (in registered mode this MUST update the registered resources).

## Outputs

### Properties

The `/properties` endpoint shows properties of the Output. In particular, it MAY contain information parsed from EDID if it is present and the Node is capable to parse EDID binaries.

### EDID

The `/edid` endpoint allows a client to download the binary EDID if it exists. If the EDID information for the Output changes and it is associated with any Receivers, then all of the Receivers in question MUST update their versions (in registered mode this MUST update the registered resources).

[IS-04]: https://specs.amwa.tv/is-04/
[IS-05]: https://specs.amwa.tv/is-05/