---
MIP: 3
Title: Universal Validator
Authors: Brian W Bush <brian.bush@iohk.io> on behalf of Alexander Nemish
Comments-Summary: No comments yet.
Comments-URI: https://github.com/input-output-hk/MIPs/wiki/Comments:MIP-0003
Status: Draft
Type: Process
Created: 2022-07-20
License: CC-BY-4.0
---


## Abstract

This document describes a modification to the token-minting process for Marlowe and the unification of the minting of role tokens and the operation of the Marlowe application or payout validators. Although it requires a change to the `Language.Marlowe.Core.V1.Semantics.MarloweParams` datatype that will eliminate some use cases for role tokens, it will enforce certain new guarantees on the role-token usage.


## Motivation

Role tokens are a primary means of authorizing deposits and choices by a party to a Marlowe contract. However, Marlowe currently has no on-chain guarantee for the minting policy of role tokens. On the contrary, any monetary policy (i.e., simple script or Plutus script) may be used for the role tokens in a Marlowe contract. The only on-chain constraint for role tokens is that all of the role tokens in a particular contract have the same monetary policy or currency symbol.

The lack of an on-chain guarantee for the role-token monetary policy opens Marlowe to several off-chain vulnerabilities: For example, a Marlowe contract might be created for role tokens having an open monetary policy that allows additional role tokens to be created after the contract commences. The controller of that monetary policy could mint duplicate role tokens for parties in the contract and impersonate them, authorizing without consent transactions using the duplicate tokens.

Thus, enforcing that each role tokens be a minted only once provides a security guarantee that duplicate role tokens will not be later minted and those duplicates used for role-based authorization of Marlowe choices and deposits. It also provides an option for minting true non-fungible native role tokens (that there is provably only ever one minted) if minting is limited to exactly one token per role.


## Specification

### Version

This change comprises a new version of the Plutus validators for Marlowe.


### `MarloweParams`

The `MarloweParams` that parameterize the universal validator must depend upon the `TxOut` consumed in the minting process, rather than the `CurrencySymbol` as is presently the case.


### Plutus Validator

The same Plutus script that validates Marlowe spending transactions must be also used to mint role tokens.


#### Minting

When used for minting role tokens, this "universal validator" must enforce the following constraints:
1.  The validator is parameterized by a `TxOut` that it consumes when minting tokens.
2. The `Redeemer` for the validator is the list of roles (i.e., token names) that will be minted, along with the number of tokens minted on a role-by-role basis.


### Spending

When used for spending UTxOs in order to advance a Marlowe contract in a transaction involving role-base authorization for a choice or deposit, the validator must compute the the `CurrencySymbol` for the role tokens from the `MarloweParams`.


## Rationale

Requiring the validator to be parameterized by a `TxOut` that it consumes when minting tokens ensures that each minting transaction creates tokens with a unique monetary policy or currency symbol.

Requiring the spending validator to compute the the `CurrencySymbol` for the role tokens from the `MarloweParams`, which in turn depends upon the `TxOut` consumed during minting, ensures that the currency symbol is unique to the *spending validator*, which also depends upon `MarloweParams`.

Thus, having the `MarloweParams` parameterize the universal validator via that `TxOut` consumed in the minting process, rather than the `CurrencySymbol`, inexorably ties the minting validator to the spending validator. The present Marlowe validator's use of `CurrencySymbol` in its `MarloweParams` allows it to accept *any* currency symbol for role tokens, but this MIP's use of the `TxOut` in the `MarloweParams` for the *universal* validator ensures that the spending validator can only use tokens minted by minting validator because both these validators are *one and the same*.

Note that the MIP forbids three potentiallly important and presently possible use cases:
1.  Customized monetary policies. *Some Marlowe dApps (e.g., DAOs or centrally administered applications) would benefit from supervisory control via a sophisticated minting policy.*
2.  Minting of additional role tokens after the original minting. *There may be certain crowd-oriented Marlowe dApps where this is desirable.*
3.  Burning of role tokens after a contract completes. *This makes it impossible ever to recover the minimum Ada associated with a role token, thus effectively increasing the up-front cost of Marlowe contracts.*


## Backwards Compatibility

This alteration to the validator is not backward compatible with the following Marlowe components:
*   Marlowe backend, `Language.Marlowe.Client`
*   Marlowe CLI, `marlowe-cli`
*   Marlowe Run, `marlowe-dashboard-server`
*   Marlowe history, `Language.Marlowe.Client.History`
*   Marlowe test cases
*   Marlowe Pioneer Program lectures


## Path to Active

In implementation is proposed in [marlowe-cardano PR#117](https://github.com/input-output-hk/marlowe-cardano/pull/117).


## Copyright

This MIP is licensed under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).
