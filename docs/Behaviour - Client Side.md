# Behaviour: Client Side

## Active Constraints of Sender

Before making an [IS-05][IS-05] connection, an NMOS Controller chooses a Sender and a number of Receivers. The NMOS Controller MAY use effective Receivers (real devices) or target Receivers (fictitious devices). The NMOS Controller MAY choose to connect the Receivers to an active Sender or an inactive Sender. If the Sender is active, the NMOS Controller MAY keep the actual configuration of the Sender or deactivate the Sender and change its configuration. If the Sender is inactive, the NMOS Controller MAY keep the actual configuration of the Sender or change its configuration.

If changing the configuration of the Sender, the NMOS Controller MUST `GET /constraints/supported` from the Sender, MUST collect Receiver Capabilities from the Receivers and make such processing of them that they can be utilized for building Active Constraints satisfying all (by default) or some (depending on user preferences) of the Receivers and using Parameter Constraint URNs supported by the Sender. If the Sender supports fewer Parameter Constraint URNs than used in the Receiver Capabilities, the NMOS Controller SHOULD inform the user about it. If the processing of the Receiver Capabilities results in an empty `constraint_sets` of Active Constraints, the NMOS Controller SHOULD inform the user about it. After that the NMOS Controller MUST `PUT /constraints/active` to the Sender and SHOULD make the IS-05 connections of the Receivers if the Constraints have been applied succesfully to the Sender.

If keeping the configuration of the Sender, the NMOS Controller MUST `GET /constraints/supported` and `GET /constraints/active` from the Sender, MUST collect Receiver Capabilities from the Receivers and make such processing of them that they can be utilized for building Active Constraints satisfying all (by default) or some (depending on user preferences) of the Receivers and using Parameter Constraint URNs supported by the Sender. If the Sender supports fewer Parameter Constraint URNs than used in the Receiver Capabilities, the NMOS Controller SHOULD inform the user about it. If the processing of the Receiver Capabilities results in an empty `constraint_sets` of Active Constraints, the NMOS Controller SHOULD inform the user about it. If the resulting `constraint_sets` is not a subset of the active constraints of the sender the NMOS Controller SHOULD inform the user about it. After that the NMOS Controller SHOULD make the IS-05 connection of the Receivers.

After breaking these connections via IS-05, NMOS Controller is RECOMMENDED to `DELETE /constraints/active` of the Sender after making it inactive.

Constraint Sets of Active Constraints SHOULD represent a consensus among Receivers. For example, there are Receiver A, Receiver B, Receiver C with equal Receiver Capabilities consisting of Constraint Sets 1, 2, 3, 4, 5 and Receiver D with Constraint Sets 2, 3, 4, 5 and 6 which is incompatible with the previous ones. In this case the consensus is Constraint Sets 2, 3, 4, 5 which are common for all the four Receivers. The user may control how the consensus is obtained, providing preferences to the NMOS Controller among the Receivers and the Constraint Sets of each Receiver and criteria about the size of the consensus.

`urn:x-nmos:cap:meta:preference` property in Constraint Sets of Receiver Capabilities indicates relative preference between Constraint Sets of that Receiver. The NMOS Controller MAY use these values to make a decision about how to process these Constraint Sets and determine preference of Constraint Sets of Active Constraints. It SHOULD NOT propagate values of this property as is from Receiver Capabilities to Active Constraints.

`urn:x-nmos:cap:meta:preference` property in Constraint Sets of Active Constraints indicates relative preference between Constraint Sets of the Active Constraints of the Sender. The NMOS Controller MAY fill in this property in Constraint Sets based on any additional information about Receivers. For example, if the user in any way informs the NMOS Controller that Receiver D has precedence over the consensus of Receivers A, B, C and `urn:x-nmos:cap:meta:preference` values of Receiver D state that Constraint Set 6 has higher preference than Constraints Sets 2, 3, 4, 5, then the Constraint Sets would be 2, 3, 4, 5, 6 where Constraint Set 6 has the highest preference.

Constraint Sets of Receiver Capabilities with `urn:x-nmos:cap:meta:enabled` set to false MUST be ignored completely while making the processing.

## Dynamic format changes on Sender

### Changes which break Active Constraints

NMOS Controller MUST track Sender's `subscription`'s `active` property during and after the IS-05 connection. If it became `false` unexpectedly, NMOS Controller SHOULD `GET /status` to figure out the reason of breaking the connection. If the State of the Sender is `active_constraints_violation`, then NMOS Controller SHOULD either disconnect all the Receivers or `PUT /constraints/active` once again and re-enable Sender via IS-05.

### Changes which do not break Active Constraints

Sender's changes which meet the Active Constraints do not affect the `active` property so if the transport file of the Sender has changed, the NMOS Controller SHOULD `PATCH /staged` of all the connected Receivers with the new `transport_file` requesting immediate activation.

[IS-05]: https://specs.amwa.tv/is-05/
