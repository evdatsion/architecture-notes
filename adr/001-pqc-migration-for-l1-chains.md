# ADR 001: Post-quantum migration path for an L1

**Status:** Accepted

## Context

Account keys on most L1s are Ed25519 or secp256k1. Both fall to Shor's algorithm on a large enough quantum computer. Nobody can give a reliable date for that machine, but three facts make waiting a bad plan:

1. Migration on an open network takes years, not months. Wallets, exchanges, custodians, bridges, indexers and hardware wallets all have to move.
2. Any account whose public key is already exposed on chain (which is every account that has ever sent a transaction) is exposed permanently.
3. NIST published FIPS 204 (ML-DSA) and FIPS 205 (SLH-DSA) in August 2024, and government guidance such as NSA's CNSA 2.0 sets transition targets across the next decade. Institutional users will start asking about this in due diligence long before the threat is real.

## Decision

1. Adopt a **composite signature** (Ed25519 + ML-DSA-65) for account keys, valid only if both halves verify. Reference implementation: [hybrid-pq-sig](https://github.com/evdatsion/hybrid-pq-sig).
2. Put a **suite byte** at the front of every key and signature encoding so new suites can be added without changing the transaction format again.
3. Roll out with a **height based policy**: legacy only, then a long overlap window where accounts rotate, then hybrid only. The enforcement height is set by governance based on the measured share of rotated active accounts, not on a fixed date.
4. Accounts rotate with a transaction signed by the legacy key that registers the composite key. The chain stores a hash of the composite key and the full key is carried only on first use.

## Consequences

- Transaction size grows by about 3.3 KB per signature. Block byte limits and fee schedules need retuning, and mempool bandwidth goes up.
- Hardware wallet support will lag. Many secure elements lack the memory for ML-DSA-65 signing today.
- If ML-DSA is broken, Ed25519 still protects accounts until a new suite ships. If Ed25519 is broken, ML-DSA protects them.
- Dormant accounts that never rotate remain classical. The chain will eventually need a policy for those (freeze, sunset, or leave exposed). That is a governance question and is deliberately not decided here.

## Alternatives considered

- **ML-DSA only.** Smaller than composite by 64 bytes, which is not the point. Rejected because the production track record of lattice implementations is still short.
- **SLH-DSA.** Very conservative security assumptions, but signatures run from about 8 KB to about 50 KB. Too big for per transaction use. Kept in mind for rare, high value actions such as governance or bridge key changes.
- **FN-DSA (Falcon).** Signatures around 666 bytes at level 1, which is attractive. Signing needs careful constant time floating point, and the standard was still being finalized when this was written. Revisit once FIPS 206 is final and implementations have some mileage.
- **Wait for a hash based account abstraction scheme.** Viable on chains with smart contract accounts, not on a chain with native accounts.
