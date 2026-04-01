---
rpip: 73
title: rETH Protection From Underperforming Nodes
description: Force exit significantly underperforming validators to protect rETH from yield loss.
author: Samus (@orangesamus)
discussions-to: <URL> TODO
status: Draft
type: Protocol
category: Core
created: 2025-07-30
requires (*optional): 44
---


## Abstract
This proposal defines the conditions under which Rocket Pool will force exit significantly underperforming megapool validators. There will be two types of issues that Rocket Pool will screen for: being mostly offline and being online with poor performance. Megapool (and minipool?) validators will be monitored and exited based on performance and time spent offline.

## Motivation
Rocket Pool node operators have been expected to perform well, since underperformance results in missed attestation penalties applied to their bonded stake and causes a loss of rewards. Despite these incentives, some node operators significantly underperform, with validators offline for extended periods of time (for example, abandoned validators). This underperformance reduces the relative value of rETH, as yield losses are socialized across all rETH holders. Historically, it has been impossible for Rocket Pool to force exit significantly underperforming validators, but with [EIP-7002](https://eips.ethereum.org/EIPS/eip-7002), it will be possible to protect rETH value by exiting significantly underperforming megapool validators. This RPIP proposes this functionality to improve rETH yield and provide the market with a more competitive liquid staking token.

## Specification

### Exit Criteria
There is consensus that we should have clearly defined rules for exits to avoid legal exposure, centralization, and scrutiny. There is no consensus yet on what such a rule set should look like, but some points regarding it were raised:
- The time frame should be long enough that one-off issues do not lead to an exit, and only operators with consistent issues are targeted.
- In practice, we see people who do not miss attestations at a high rate but are missing out on a significant share of rewards because attestations are late. Therefore, a rule that simply exits based on missed attestations appears insufficient.
- Another case that we may want to cover is someone attesting completely fine but missing all block proposals (for example, because of an MEV-Boost issue).
- One suggestion is to start conservatively and raise exit thresholds over time to gauge how this plays out in practice.

### Trustless System
It appears possible to implement a trustless system for exiting offline validators by utilizing a beacon state proof about validator balance.

The team is still exploring if a proof-based system is possible for underperforming (but online) validators using epoch participation in the beacon state. The beacon state has all the information about participation, correctness, and timeliness of source, target, and head, including for historical blocks (historical_summaries). To monitor attestation performance and not just raw attestation correctness, a challenger could provide epoch lists for different categories (no attestation, late head vote, etc.) that together imply that the performance is below a chosen threshold. The NO can defeat an incorrect challenge by providing a proof that one of the epochs included by the challenger does not meet the criteria. After the challenge period has passed, the validator can be force-exited. The epoch lists could be provided in calldata and merkleized to make gas costs feasible (3 million gas in calldata would cover roughly 100 days’ worth of epochs, and only a subset of epochs would need to be listed to show underperformance).

For underperformance, using the validator balance at two different points in time alone appears challenging, since assigned duties (block proposals and sync committees) lead to variable returns.

Implementing the trustless system for offline validators but not underperformance, or vice versa, may not be a high priority, since a mixed system would have the same trust assumptions as a fully trusted one.

### Exiting Entities
If we are not able to fully enforce the rule set on-chain, an entity will have to be trusted to initiate exits according to the rule set. Consensus seems to be that this should be the oDAO, but we would be looking to have safeguards. This could be in the form of a security council or the pDAO needing to sign off or having veto ability.

For the pDAO, two approaches seem feasible:
- If the oDAO proposes a list of nodes/validators to exit, this automatically triggers a full pDAO vote to decline this list. This means that the default outcome (if quorum is not reached) is that the list will be exited, but any exit is delayed by a full vote period.
- After the oDAO submits a list, a challenge period starts during which anyone can provide a bond to trigger a veto vote. If the veto fails, the bond is lost. The bond prevents someone on the exit list from delaying their exit by the vote period. The challenge period can be shorter than a full vote period, so this approach would lead to less delay before exits, but the risk of losing a bond might prevent people from challenging where there are legitimate but minor issues with the proposed exit list.

### Minipools and Old Delegate
Force exiting minipools would require a new minipool delegate and the node operator to upgrade to that delegate or use the latest delegate set. Because node operators that are below 32 ETH effectively get 100% commission, and because underperformers may still be more profitable than solo staking, node operators would have an incentive to avoid such a new delegate.

One suggestion to align node operator interests with protocol interests is to apply penalties to minipools that meet exit criteria but are not able to be exited because of their delegate version. Such a penalty should be large enough to make exiting and losing out on future rewards less impactful. Node operators would then benefit from being on the new delegate or voluntarily exiting in case they meet exit criteria. This idea and penalty sizing need more discussion.

### Voluntary Exiting before Force Exit
Whenever we are about to force exit a node/validator via EIP-7002, we prefer the node operator to exit via the regular exit mechanism because there is no gas cost or fee, and the number of EIP-7002 exits per block is limited.

In the case of oDAO-trusted exits, the pDAO vote or challenge period provides a natural window for node operators to do this. The discussed trustless system for underperformance has a challenge period as well. The trustless system for being offline does not provide this opportunity.

A penalty that comes with EIP-7002 exits would give node operators an incentive to preempt it with a voluntary exit.

### Watchtower Automation
The specification is compatible with an automated watchtower process submitting nodes/validators to exit, but does not require it. This gives flexibility to start out with a more manual process and prevents watchtower implementation from becoming a blocker.

### Exiting Entire Node vs Specific Validators
Validator issues tend to be correlated to the node, but there are cases where issues may be limited to only some validators (migrated validators with different seed phrases, an RP node with validators on more than one machine), so the ability to exit specific validators may be useful.



## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
