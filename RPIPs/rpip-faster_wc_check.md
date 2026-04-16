---
rpip: <to be assigned>
title: Faster Withdrawal Credentials Check
description: Replace validator beacon state proofs with deposit contract root fraud proof system to protect against malicious withdrawal credentials.
author: knoshua (@knoshua)
discussions-to: <URL>
status: Draft
type: Protocol
category (*only required for Protocol ): Core
created: 2026-04-01
---

## Abstract
This RPIP proposes to extend the beacon state proof mechanism that protects megapool validator creation from withdrawal‑credential front‑running with a fraud‑proof system based on the beacon chain deposit contract root.
Instead of requiring a beacon state proof for every new megapool validator, the protocol would only require a proof in the event of a withdrawal‑credential exploit.

## Motivation
The Saturn 1 upgrade removed the oDAO scrub check duty and replaced it with a beacon state proof about the validator. Since EIP-7251 (implemented in Electra), the deposit signature is validated and the pubkey is compared
to existing validators only after a validator makes it through the `pending_deposits` queue. This means that the state proof, and therefore the second deposit (31 ETH) for a validator can only happen once the first
deposit (1 ETH) has made it through. This proposal speeds up this process and remove the dependency on the entry queue. Additionally, proofs would only need to be submitted in case of an attempted withdrawal credential
exploit and, in the common case of honest node operators and correct withdrawal credentials, no proof is required.

## Specification
This specification introduces the following pDAO protocol parameters:

| Name                        | Type  | Initial Value | Guardrail |
| --------------------------- | ----- | ------------- | --------- |
| `prestake_challenge_period` | Hours | `12`          | `≥12`     |

- During the `prestake` transaction for a megapool validator and before the actual deposit to the deposit contract, the Merkle root of the Beacon Chain deposit contract’s deposit tree SHALL be stored. 
- After `prestake` has been called, anyone SHALL be able to provide a Merkle proof of inclusion plus the full deposit data and signature that the stored root contains a deposit for the same BLS validator pubkey. The signature of the earlier deposit SHALL be validated.
- If such a proof is submitted, the rETH share of the remaining 31 ETH SHALL be returned to the deposit pool and the remaining node operator share SHALL be awarded to the provider of the proof. [TODO: Consider sending everything to the deposit pool and giving challenger credit or rETH.]
- If no proof is provided within `prestake_challenge_period`, anyone SHALL be able to call `stake_without_state_proof`, which SHALL stake the remaining 31 ETH for the validator.

## Rationale

The BLS12-381 signature of the validator that is part of a deposit is not verified by the deposit contract, and it is also not verified before a deposit is added to `pending_deposits`. As a result, anyone could
“fake front-run” a legitimate validator with a deposit of 1 ETH using the same pubkey together with a fake signature. Such a deposit would be part of the deposit root, so the signature has to be validated to avoid a
griefing vector where someone could spend 1 ETH to have legitimate Rocket Pool validators dissolved.

The existing beacon state proof system remains in place and allows short-circuiting the challenge period in cases where there is no beacon chain queue.
[TODO: What are maintenance costs for keeping it around and is that actually worth the minor benefit?]

With the proposed specification, any deposit (with valid signature) at time of prestake results in a dissolve. This hypothetically includes deposits that did set the correct withdrawal credentials and
would not actually cause an issue, because there is no legitimate reason for such deposits to exist. This is a conscious choice to not leave any room for unforeseen exploits.

We considered using `pending_deposits` in a fraud-proof system instead of the Beacon Chain deposit contract. While it may initially be easier to implement a proof generator against the beacon state,
these proofs are not stable and may require contract updates with future Ethereum forks. The Beacon Chain deposit contract approach appears more stable and will likely not require maintenance work going forward.
The initial extra work in building a proof generator therefore seems worthwile in the long run.

## References
- [EIP-2537](https://eips.ethereum.org/EIPS/eip-2537) introduced precompiles for BLS12-381 curve operations that allow signature verification.

## Security Considerations [TODO]

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
