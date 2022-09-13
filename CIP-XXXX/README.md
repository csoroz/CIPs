---
CIP: ?
Title: On-chain governance committee membership replacement
Authors: Duncan Coutts <duncan.coutts@iohk.io>, Jared Corduan <jared.corduan@iohk.io>
Comments-URI: https://github.com/cardano-foundation/CIPs/wiki/Comments:CIP-TBD
Status: Draft
Type: Standards Track
Created: 2022-07-28
License: CC-BY-4.0
---

# Abstract

This CIP extends the Cardano ledger rules relating to governance with two
features:

1. A new on-chain mechanism to enable the committee of "master key" holders to
   agree to replace themselves with a new committee.
2. A change to the existing on-chain rules to require protocol parameter updates
   that schedule a hard fork to be signed by a threshold of the master key
   holders and a threshold of their delegated governance keys -- whereas
   previously only the threshold of delegated governance keys was required.

# Motivation

Recall that the simple centralised governance system that was introduced
in the Cardano Shelley era works like this:

* There are 7 original "master keys", as specified in the original Byron genesis
  configuration.
* The original master keys are held by IOG, the Cardano Foundation and Emurgo.
* These master keys delegate to 7 corresponding "governance keys".
* The delegation from each master key to its corresponding governance key can be
  updated on-chain at any time, using a delegation certificate signed by the
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
minimally modified to hand governance to a new committee -- ideally with broad
community support and legitimacy -- which can then through off-chain legal and
community means establish agreement and consent for a final decentralised
governance system, to be enacted via a later hard fork. Thus this CIP proposes
two minimal changes that would allow the existing centralised governance to be
transferred to a new committee, and used safely.

The first change allows the committee holding master keys to vote on-chain to
replace itself. This would require signatures from a minimum threshold of the
existing committee. The new master keys committee is specified on-chain as a
new set of keys, and a new threshold for how many signatures will be required
for decisions in the new committee. Note that this allows changing the size of
the committee. At the point when this feature is introduced at a hard fork, the
initial committee would consist of the original 7 master keys, with the existing
threshold of 5 signatures required.

This first feature is required because the existing centralised governance
system does not have any way for the master keys to be replaced, only to change
the delegated governance keys.

The existing centralised governance involves a notion of delegation from each
master key to a governance key. It would be useful to have the flexibility to
use this delegation between keys to allow delegation between different
individuals holding the keys, acting in different roles, and to do so while
only requiring limited trust. For example, an individual or organisation holding
a master key may have clear community support to act in a governance oversight
role, but not themselves have the technical capability or the time to also act
in the operational governance role. Another example might be that one
organisation might hold many master keys and delegate to a set of governance
keys held by other individuals. This provides a simple approximation of M:N
delegation. There may be other useful examples. The point is simply that such
flexibility would likely be useful, but it is important that such flexibility
can be exercised safely. In the existing centralised governance system however,
the delegation from master key to governance key implies full trust, because (a
threshold of) the governance key holders can invoke hard forks which can change
anything, including the governance system.

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

This second feature is required to allow the holders of the master keys and the
holders of the delegated governance keys to be different groups of individuals,
without also allowing the holders of the delegated governance keys to act
unilaterally to change any aspect of the system (including governance).

# Specification

Both features described above in the motivation section can be implemented by
modifying the current protocol parameter update system.

## New protocol parameters

Two new protocol parameters are added:

| Field      | Type                           | Description           |
| -----      | ----                           | -----------           |
| masterKeys | set of verification key hashes | the master keys       |
| quorum     | unit interval rational         | the master key quorum |

We require that the quorum value be greater than one half so that if quorum is
met the decision is unambiguous.

## New master keys

Whenever new master keys are adopted as a part of the usual protocol parameter
update changes in the `NEWPP` ledger transition, the mapping from master keys to
governance keys is modified so that:

* the domain of the map is the new set of master keys
* the range of the map (the governance keys) all have a nullary value

This map is named `genDelegs` in the
[Shelley Ledger Specification](https://hydra.iohk.io/job/Cardano/cardano-ledger/shelleyLedgerSpec/latest/download-by-type/doc-pdf/ledger-spec)
and is a part of the `DState` record type.
Note that `DState` is currently a part of the `NEWPP` environment, so it will
have to be moved to the `NEWPP` state.
The `EPOCH` rule (which calls the `NEWPP` rule) must adjusted to accommodate
this change (though the `EPOCH` rule already mutates the `DState`).

## Immediate master key delegation

Delegating a master key to a governance key is currently delayed by one
stability window (twelve hours on mainnet) after posting a genesis delegation
certificate, due to the header only validation done by the consensus layer.
See section 15.1 in the
[Shelley Ledger Specification](https://hydra.iohk.io/job/Cardano/cardano-ledger/shelleyLedgerSpec/latest/download-by-type/doc-pdf/ledger-spec)
for an explanation of how header only validation effects the ledger.

Because the Vasil hardfork removed the overlay schedule, however,
this delay is no longer necessary: the only way that the governance keys can
effect header validation is by their ability to authorize the overlay blocks.

Therefore the staged master key map, named `fGenDelegs` in the `DState` type,
can be removed, together with all logic associated with it.
Moreover, the `Deleg-Gen` rule of the `DELEG` transition will now immediately
update the `genDeleg` map.

## Authorization scheme

The current authorization scheme for protocol parameter updates is:

* 5 of 7 **governance** keys must authorize
  (sign the transaction ID)
  any transaction which includes a protocol parameter update.

This logic is done in the `UTXOW` transition.
In what follows, we will say "quorum must be met" to mean that
`ceiling (quorum * |masterKeys|)`-many participants authorize a change,
whether they be master keys or governance keys.

The new authorization scheme will be:

* quorum must be met by the **master** keys regarding any protocol parameter
  updates which propose to change any of the following protocol parameters:
    - the master keys
    - the quorum
    - the major protocol version
* quorum must be met by the **governance** keys regarding any protocol parameter
  updates propose to change of any of the protocol parameters not listed above

Note that both the master keys and the governance keys could be required to
authorize a protocol parameter update.

## Replace the `quorum` global constant

All uses of the `quorum` global constant will be replaced with the new `quorum`
protocol parameter:
- the authorization of protocol parameter updates (described above)
- the authorization of MIR certificates

# Rationale

We aim to provide minimal changes that enable the existing governance mechanism
to be safely transferred, without needing future hardforks (beyone the one that
introduces these changes).

The rotation mechanism could have been implement in several different ways,
such as a new delegation certificate or a new field in the transaction body.
The existing protocol parameter update system, however, already exhibits the
desired behavior and is the most natural way to achieve the goals.
Credit goes to Andre Knispel for this observation.

# Backwards compatibility

This change is not backwards compatible; it requires a hardfork.
The only change to the CBOR specification is the addition
of the two new protocol parameters.

# Path to Active

A hardfork is required for these changes.
A new ledger era is needed, containing the changes described.
No changes are needed for the consensus or networking layer,
but the CLI will need to support the new protocol parameters.

# Copyright

This CIP is licensed under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).
