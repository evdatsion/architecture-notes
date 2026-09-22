# ADR 002: Treat consensus keys and account keys as separate migrations

**Status:** Accepted

## Context

A BFT chain signs two very different kinds of things:

- **Account signatures** on transactions. One per transaction, verified once, stored in blocks.
- **Consensus signatures** from validators on prevotes, precommits and proposals. Every validator signs every round, and commits carry a signature from each validator that took part.

With 100 validators and a 3.3 KB composite signature, one commit carries about 330 KB of signatures. That goes into every block, into light client headers, and into IBC style cross chain proofs. The pressures on the two key types are different enough that one decision for both would be wrong for at least one of them.

## Decision

1. Account keys follow ADR 001.
2. Consensus keys get their own track and their own timeline. Evaluate:
   - aggregation friendly post-quantum schemes as the research matures,
   - committee based commits where only a sampled subset signs each block, verified against a rotating committee root,
   - a hybrid where classical BLS aggregate signatures carry liveness and a periodic post-quantum checkpoint signature carries long term safety.
3. Until a consensus scheme is chosen, keep validator keys classical but make the validator key type a registered, versioned field so the switch is a parameter change plus a coordinated upgrade, not a data migration.

## Consequences

- Short term, consensus stays classical. The realistic attack there is forging validator votes, which needs a quantum adversary acting live, while account keys are exposed forever once published. Accounts go first for that reason.
- Light client and bridge designs need to be ready for a future header format with larger or different signatures.

## Alternatives considered

- **Same composite scheme for both.** Simple, but it makes every commit and every light client proof roughly 50 times bigger. Rejected for chains with more than a handful of validators.
