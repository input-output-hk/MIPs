---
MIP: ???
Title: Input Commitment Protocol
Authors: paluh <tomasz.rybarczyk@iohk.io>
Comments-Summary: No comments yet.
Comments-URI: https://github.com/input-output-hk/MIPs/wiki/Comments:MIP-???
Status: Draft
Type: Standards
Created: 2024-01-07
License: CC-BY-4.0
---


## Abstract

This document describes a simple protocol which introduces a delegetion scheme of the `Input`s from a role to another independent party. The party gains a priviledge to perform a particular step or set of steps in the contract on behalf of the delegator. The scheme is minimal by design so the acompanying Plutus script is tiny and easy to audit.
We don't limit the `Input` type to an `IChoice` because this scheme allows us to sign chain of `Input`s even though some of them could be performed by other party. In many cases the `IChoice` itself will be the commitment but it could come with a bundle of "conditions".

The protocol does not provide execution guarantees on its own - those should be implemented using different Marlowe patterns on the contract level. I will provide an illustrative example.


## Motivation

Delegating an execution of an action to another party may seem not particularly useful at a first glance. But if we look at it as an irreversible commitment from the delegator side and if we also add the transparent and deterministic nature of Marlowe to the picture we can start seeing a possibility for more powerful protocols which can be built on top of that.

As a motivating example I would like to introduce a draft of unidirectional payment channel implementation with entirely offchain payment process till the final cash out. The channel will be fully secure and not bounded by the number of atomic transfers but by the limit of the funds locked in it. In other words we will be able to compress many off-chain subpayments into a single on-chain one.

```mermaid
sequenceDiagram
    participant Sender
    participant Cashier Plutus Script
    participant Recipient
    participant Marlowe Payment Channel
    Recipient-->>Sender: 5 ADA bill
    Sender-->>Recipient: 5 ADA cheque
    Recipient-->>Sender: 7 ADA bill
    Sender-->>Recipient: 12 ADA cheque
    Recipient->>Marlowe Payment Channel: I want to cash out
    Recipient->>Cashier Plutus Script:12 ADA cheque
    Cashier Plutus Script->>Marlowe Payment Channel:12 ADA cheque is OK
    Marlowe Payment Channel->>Recipient:12 ADA payout
```

So in the scheme above the dashed arrows (`-->`) represent off-chain communication. Please note that after the second bill `Sender` just issues a new cheque which covers both expenses. Because there will be only a single payout opportunity the highest one will be used and the smaller ones will be discarded.

The solid arrows are parts of the final cash out transaction. They are rather conceptual and they don't directly represent element of the final contract which is sketched below:


```mermaid
flowchart TD
    A(When) -->|Deposit by Sender| B{When}
    B --> |IChoice 'cash out' by Recipient|C(When)
    B --> |IChoice 'closing channel' by Sender| D(When)
    B --> |Timeout far away in the future|G(Close)
    C --> |IChoice 'amount' by Cashier| E(Pay 'amount' to Receipient)
    D --> |IChoice 'cash out' by Receipient|F(When)
    D --> |Contestation timeout in the near future|H(Close)
    F --> |IChoice 'amount' by Cashier| I(Pay 'amount' to Receipient)
    I --> K(Close)
    E --> J(Close)
```

In the contract above the `Cashier` action requires commitment from the `Sender` side. So `IChoice "amount"` is the cheque which `Sender` and `Recipient` exchange off-chain and this signed `Input` is used at the end to cash out the money on the L1 layer.
We can use two different keys for `Sender` wallet and `Sender` commitments so the application which will issue the cheques doesn't even have to be the wallet itself.

The "cash out" branch is guarded by `Recipient` action so the cheques are safe and cannot be used by `Sender` to initiate the payout process with outdated cheque. The cheques themselves can be stored even publically or send to multiple storage backends for redundancy (email, IPFs etc.).

We can even consider that separation of wallets from the payment channel App and use separate key for `Recipient` from the final destination wallet. This separation could allow us to minimize the risk but also to minimize the communication with the wallet from the application (possibly a mobile App).

## Specification

### Version

This change comprises the Plutus validators which implements input commitment.

## Rationale

> The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work. The rationale should provide evidence of consensus within the community and discuss important objections or concerns raised during discussion.|

## Backwards compatibility

> | All MIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The MIP must explain how the author proposes to deal with these incompatibilities.|

## Path to Active

> | Path to Active | A reference implementation, observable metrics or anything showing the acceptance of the proposal in the community. It must be completed before any MIP is given status "Active", but it need not be completed before the MIP is accepted. It is better to finish the specification and rationale first and reach consensus on it before writing any code. |

## Copyright

This MIP is licensed under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).
