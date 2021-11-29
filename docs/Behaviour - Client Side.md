# Behaviour: Client Side

## Constraints of Sender

Before making an [IS-05][IS-05] connection, NMOS Controller chooses a Sender and Receivers. Then NMOS Controller MUST `GET /constraints/supported` from the Sender, collect Receiver Capabilities from the Receivers and make such intersection of them that they can be utilized for building Constraints satisfying all the Receivers and using Parameter Constraints supported by the Sender. If the Sender supports less Parameter Constraints than used in the Receiver Capabilities, the NMOS Controller SHOULD inform the user about it. After that NMOS Controller MUST `PUT /constraints/active` and make the IS-05 connection if contraints have been applied succesfully. After breaking this connection via IS-05, NMOS Controller is RECOMMENDED to `DELETE /constraints/active`.

If multiple Receivers have equal Constraint Sets, NMOS Controller MUST take only one of them. If they differ only in `urn:x-nmos:cap:meta:preference`, then one with the highest priority MUST be taken.

## Dynamic format changes on Sender

### Changes which break Constraints

NMOS Controller MUST track Sender's `subscription`'s `active` property during the IS-05 connection. If it became `false` unexpectedly, NMOS Controller SHOULD `GET /constraints/active` to figure out the reason of breaking the connection. If the request returns `205 Reset Content`, NMOS Controller SHOULD either disconnect all the Receivers or `PUT /constraints/active` once again and re-enable Sender via IS-05.

### Changes which do not break Constraints

Sender's changes which follow the Constraints do not effect the `active` property so NMOS Controller MUST track version of Inputs, the Source, the Flow and the Sender. If any of them has changed, NMOS Controller SHOULD `PATCH /staged` of all the Receivers with new `transportfile` if it's used for the IS-05 connection.

[IS-05]: https://specs.amwa.tv/is-05/