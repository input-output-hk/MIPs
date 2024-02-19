---
MIP: ???
Title: Timeout Continuation Considered Harmful
Authors: paluh (aka Tomasz Rybarczyk) <t.rybarczyk@lambdaterms.com>
Comments-Summary: No comments yet.
Comments-URI: https://github.com/input-output-hk/MIPs/wiki/Comments:MIP-0003
Status: Draft
Type: Standards
Created: 2024-01-07
License: CC-BY-4.0
---

## Abstract

This proposal introduces a substantial simplification to the Marlowe smart contract language structure by suggesting the removal of `timeout_continuation` while retaining the `timeout` mechanism. Currently, Marlowe allows contracts to specify actions upon timeouts, but this proposal argues that such continuations are largely redundant and can be effectively replaced with a combination of `Notify` actions and a straightforward closure of contracts upon timeout.

This change aims to streamline the language changes by eliminating unnecessary complexity and enabling introduction of new construct `WhenAll`. Moreover, it advocates for embracing runtime exceptions as part of the language's semantics, moving away from complex and potentially hazardous defaulting logic that introduces risks to contract integrity and execution.

The proposal outlines several advantages of this approach, including a cleaner semantic model, the facilitation of new features like `WhenAll`, and improvements to the contract merkleization process. By redefining the treatment of timeouts and exceptions, this proposal seeks to make Marlowe contracts more accessible, secure, and powerful.

## Motivation

### Accidental Complexity Imposed by `timeout_continuation`

In the current framework of the Marlowe protocol, `When` clauses serve as essential mechanisms for orchestrating user interactions or managing timeouts. However, this design is significantly encumbered by the `timeout_continuation` mechanism in non obvious way. 
Intended to safeguard against the locking of funds by ensuring contract progression, it introduces an unintended consequence - the requirement that the contract can always progress to its conclusion, even in the face of erroneous situations.

I think that the most important language properties which `timeout_continuation` enforces on us are: 

* **Forced Continuity Through Defaulting Logic**: The protocol's requirement for uninterrupted progression to a `Close` operation—essential for the release of funds without risking their indefinite lock-up—mandates the implementation of defaulting logic for situations like division by zero, undefined variables or negative money transfers. This necessity introduces a vector for not obvious outcomes, undermining the protocol's reliability.

* **Obstacles to Innovation**: The desire to introduce new constructs to the language as `WhenAll` (we could rename the current `When` to `WhenAny`), which would allow contracts to pause until all specified inputs are received, is significantly constrained by the existing `timeout_continuation` branch. `timeout_continuation` branch implies that after the `WhenAll` branch we end up with coherent state of the contract which actually not true because this branch will be actually fired if *any* of the inputs was not provided before the timeout!

* **Acknowledging the Reality of Exceptions**: The protocol's current stance on exceptions fails to fully account for the intrinsic resource limitations of any execution environment (in our case we have pretty severe blockchain limits). By recognizing these limitations and incorporating them into Marlowe's semantics together with other exceptions, we can simplify the language and enhance the predictability and safety of it.


By acknowledging the undue burden `timeout_continuation` places on the language I underscore the need for reevaluation.


## Marlowe with Plain `timeout`

In response to the challenges posed by `timeout_continuation`, this proposal advocates for a streamlined approach: retaining only the timeout mechanism and adopting `Close` as the default action upon reaching this timeout. This simplification preserves the core functionality of timeouts—mitigating the risk of fund locking—without the accompanying complexity burden. By narrowing the context of timeouts to a single `Close` action, we can focus on making this action as robust and predictable as possible, thereby enhancing the overall safety and reliability of Marlowe contracts. In the current setup it is still pretty easy to lock the funds even in single `Close` (it is enough to have 10 to 12 accounts!).

The current use of `timeout_continuation` is also identified as largely redundant. It can be more effectively and transparently managed through the use of `Notify` in conjunction with `TimeIntervalStart` based condition, which more naturally align with the rest of the language. This shift not only simplifies the language's but also exposes the users to the utility of `Notify` construct which could be used in other contexts as well.

### Semantic Difference Between `timeout_continuation` and `Notify`


As state above the `timeout_continuation` construct can be effectively replaced with `Notify (TimeIntervalStart > timeout)` condition followed by the desired continuation. The diffence in semantics is that `Notify` based `timeout` should be "triggered" by interested user before the actual `timeout` of the enclosing `When` is reached. Current `timeout_continuation` when reached is not capped by any other timeout condition.

I would argue that this trigger based approach is pretty natural and consistent with the of the language - all inputs continuations are picked if an interested party provides triggers it on time and in other cases we naturally expect the contract to close. By embracing a `Notify`-based approach to timeouts, we foster an interaction model which we already have in place.

## The Possible Benefits of The `timeout_continuation` Removal


I want to expand on the possible far-reaching impacts of simplifying the language. This isn't just about making the language easier to use; it's also about how it can lead to easier language semantics, better safety and efficiency on the blockchain.

### Highly Constrained Timeout Environment

The proposed simplification clearly narrows down the execution context of the timeout procedure. It lets us concentrate on enhancing the `Close` construct's reliability and might even allow us to relax how we handle errors elsewhere. The `Close` reliability can be improved on the language level by introduction of lazy and on-demand payout loop evaluation. This could solve a critical problem of fund locking which exists currently on the chain. I will expand on that in a separate proposal.

The freedom we get from this tighter timeout scenario opens new doors. We're now in a position to rethink our error management strategy, acknowledging errors more openly and being upfront about them. It means we can outright reject any execution branch that hits a runtime error or gobbles up too much resources (which, in itself, is a form of runtime error). We can avoid non obvious evaluation strategies for clearly invalid operations like division by zero, undefined variable usage or negative deposits. This in turn would increase clarity of the language semantics. Plus, we get to officially include the `Assert` construct in our language toolkit, turning it into a possible proper control flow element.

It's important to note that this new stance on exceptions is viable because a contract that raises exceptions before reaching the first `When`—a safe point ensuring the non-locking timeout guarantee—cannot lock funds. This is because, prior to the first `When`, the account state must be empty since no deposits have been made yet (`Deposit` actions occur only within the `When` construct).


### Introduction of `WhenAll`

A really well know limitation of the current `When` construct is it's inability to clearly encode the "await all inputs" semantics. This has far reaching practical consequences. For example to encode a series of deposits which can happen in an arbitrary order we are focred to build a contract whose size increases exponentially with the number of deposits.

Previous efforts to introduce a `WhenAll` construct were hindered by the `timeout_continuation` mechanism. If we were to add a `WhenAll` construct similar in structure to the current `When`, its `timeout_continuation` branch would activate if **at least one** input was missing before the timeout, resulting in an incoherent execution environment and complicating the continuation contract design.

Simplifying the timeout mechanism paves the way for the straightforward inclusion of `WhenAll`, without necessitating further language modifications. A timeout branch with simple `Close` semantics, executing the payout loop, resolves this ambiguity. We only need to consider the account state, which might be partially updated by a subset of `WhenAll` deposits, as choices and any other parts of the contract state are disregarded by the `Close` semantics.
`WhenAll` and possible modification of `When` called `WhenAny` will be a topic of a separate proposal.

### Consistent Merkleization

The current `timeout_continuation` contract is not merkleized in the current setup because we want to guarantee that the contract will always progress to the final `Close`. In practice we used artificialy `Notfiy` interleaved encodings to enforce timeout branch merkleization which is not natural and hard to understand for the users. By replacing `timeout_continuation` with a default `Close` sematics, we avoid the need for merkleization of that branch. Additionally the proposed `Notify` based timeout mechanism is merkleized in the same way as any other contract continuation.


### Delayed Payout Example 

This is more anectodal observation but probably still worth mentioning. A common introductory example which we provide to the users is a simple  contract which contains only deposit after which we enforce a timeout so the payout is executed after a enforced delay. Fortunatelly the proposed change would not affect this scenario at all. The `Close` branch would be executed by default after the timeout and the payout would be executed as expected. 

## Specification

The only change implied by this proposal is the removal of the `timeout_continuation` from the `When` the construct and usage of `Close` semantics upon the timeout.

## Backwards compatibility

Unfortunately this change is not backwards compatible in obvious way and would require of migration of all the Marlowe implementations which we maintain.

Because of that it should be considered as a part of a larger change which would include also introduction of `WhenAll` and `WhenAny` constructs, change in the exception handling and introduction of the lazy payout loop evaluation.

### Version

The implementation is not provided yet. It will be included if the proposal is received positively.

## Copyright

This MIP is licensed under [CC-BY-4.0](https://creativecommons.org/licenses/by/4.0/legalcode).
