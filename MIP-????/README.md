---
MIP: ????
Title: Composable Payment Channels Protocol
Authors: paluh <tomasz.rybarczyk@iohk.io>
Comments-Summary: No comments yet.
Comments-URI: https://github.com/input-output-hk/MIPs/wiki/Comments:MIP-????
Status: Draft
Type: Standards
Created: 2024-01-08
License: CC-BY-4.0
---


## Abstract

This document describes an extension over `Inputs Commitment` proposal which allow cross chanel composition of commitments. Composition of commitments can be seen as a simple cross chanel consensus protocol which we can achieve easily by introducing signature dependency.
A party which controls intermediate payment channel in the chain of should require a signed cheque from the payer side with additional fee to issue a cheque for the recipient. So in general the chain of payments could be initiated from the buyer to the seller but if we don't tie this up with extra dependency on the seller signature the buyer can loose money. I will provide more detailed example in the next secions.

## Motivation

> | Motivation | The motivation is critical for MIPs that want to change the Marlowe protocol. It should clearly explain why the existing protocol is inadequate to address the problem that the MIP solves. |

## Specification

> | Specification | The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Marlowe platforms. |

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
