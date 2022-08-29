---
CIP: ?
Title: On-chain governance committee membership replacement
Authors: Duncan Coutts <duncan.coutts@iohk.io>
Comments-URI: https://github.com/cardano-foundation/CIPs/wiki/Comments:CIP-TBD
Status: Draft
Type: Standards Track
Created: 2022-07-28
License: CC-BY-4.0
---

# Abstract

This CPI extends the Cardano ledger rules relating to governance with two
features:

1. Enabling the committee of "master key" holders to agree to replace themselves
   with a new committee.
2. Requiring protocol parameter updates that schedule a hard fork to be signed
   by a threshold of the master key holders, rather than just a threshold of
   their delegated governance keys.

# Motivation

Recall that the simple centralised governance system that was introduced with
in the Cardano Shelley era works like this:

* There are 7 original "master keys", as specified in the original Byron genesis configuration.
* The original master keys are held by IOG, the Cardano Foundation and Emurgo.
* These master keys delegate to 7 corresponding "governance keys".
* The delegation from each master key to its corresponding governance key can be
  on-chain updated at any time, using a delegation certificate signed by the
  master key.
* The governance keys are used to sign protocol parameter updates (including
  changes to the major protocol version, which schedules a hard fork) and
  transfers from the treasury or reserves.
* A threshold of 5 of the 7 governance keys is required to sign protocol updates
  or treasury/reserves transfers.

The Shelley centralised governance system was always intended to be replaced
eventually as part of decentralising the governance of Cardano. There is however
somewhat of a "chicken and egg" difficulty with introducing decentralised
governance: one would ideally like to use a decentralised governance system to
get proper agreement and consent to a "constitutional" change that introduces
decentralised governance!

To side-step this difficulty, the existing centralised governance could be
minimally modified to hand governance to the Cardano Foundation, which can then
through off-chain legal and community means establish agreement and consent for
a final decentralised governance system, to be enacted via a later hard fork.
Thus this CIP proposes two minimal changes that would allow the existing
centralised governance to be transferred to a new committee, and used safely.

The first change allows the committee holding master keys to vote on-chain to
replace itself. This would require signatures from a minimum threshold of the
existing committee. The new master keys committee is specified on-chain as a
new set of keys, and a new threshold for how many signatures will be required
for decisions in the new committee. Note that this allows changing the size of
the committee. At the point when this feature is introduced at a hard fork, the
initial committee would consist of the original 7 master keys, with the existing
threshold of 5 signatures required.

This feature is required because the existing centralised governance system does
not have any way for the master keys to be replaced, only to change the
delegated governance keys.

The second feature prevents the holders of the delegated governance keys from
"overthrowing" the holders of the master keys. This would otherwise be possible:
the governance keys can sign protocol parameter updates; the protocol parameters
includes the major protocol version which controls hard forks; hard forks can
in principle change any system rules, including governance rules. The normal
rule is that protocol parameter updates must be signed by at least a threshold
of the delegated governance keys. The proposed modified rule is that protocol
parameter updates that modify the major protocol version must additionally be
signed by the same threshold of the master keys. This ensures that there is
consent from the master key holders for all hard forks, which may include
governance changes.

# Specification

# Rationale

# Backwards compatibility

# Path to Active

# Copyright

This CIP is licensed under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).
