# architecture-notes

Working notes and architecture decision records from R&D across blockchain infrastructure, verifiable AI and post-quantum cryptography.

Most of this started as internal write-ups at [Magnus Mage](https://magnusmage.com) and while building [Xyress](https://xyress.com). Client specifics are stripped out. What is left is the reasoning, which tends to be the part people ask about anyway.

Nothing here is a tutorial. Each note assumes you already know the basics and want the trade-offs.

## Decision records

| ADR | Title | Status |
|---|---|---|
| [001](adr/001-pqc-migration-for-l1-chains.md) | Post-quantum migration path for an L1 | Accepted |
| [002](adr/002-separate-consensus-and-account-keys.md) | Treat consensus keys and account keys as separate migrations | Accepted |
| [003](adr/003-anchor-roots-not-records.md) | Anchor AI audit roots on chain, keep records off chain | Accepted |

## Notes

| Note | Topic |
|---|---|
| [crypto-agility.md](notes/crypto-agility.md) | Designing transaction and key formats that can change algorithms without a hard fork |
| [bft-in-production.md](notes/bft-in-production.md) | What breaks first when you run a Tendermint style BFT chain for real |
| [verifiable-ai-stack.md](notes/verifiable-ai-stack.md) | Receipts, TEEs and zkML, and which one a given use case actually needs |

## Related code

- [hybrid-pq-sig](https://github.com/evdatsion/hybrid-pq-sig): composite Ed25519 + ML-DSA-65 signatures with a height based migration policy
- [inference-anchor](https://github.com/evdatsion/inference-anchor): Merkle batched receipts for AI inference

## Format

ADRs follow the usual shape: context, decision, consequences, alternatives. A superseded ADR stays in place with a pointer to the one that replaced it. Being able to see why we changed our minds is half the value.
