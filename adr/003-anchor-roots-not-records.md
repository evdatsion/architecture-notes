# ADR 003: Anchor AI audit roots on chain, keep records off chain

**Status:** Accepted

## Context

We need AI inference records that the operator cannot rewrite after the fact, for customer disputes, regulator requests and internal incident review. The naive approach is writing each record to a chain. It is expensive, it leaks metadata (volume, timing, sometimes content), and it ties the AI serving path to chain latency.

## Decision

1. Each inference produces a receipt of hashes and metadata: model digest, salted input and output commitments, sampling parameters, timestamp.
2. Receipts are batched into an RFC 6962 style Merkle tree. Only the root and the batch size are anchored.
3. Callers get their receipt and inclusion proof in the API response.
4. Anchoring is asynchronous. Serving never waits on the chain. Batches close on size or age, whichever comes first.
5. The anchor interface is one method, so the anchor can be a public chain, a permissioned chain, or a transparency log depending on the deployment.

Reference implementation: [inference-anchor](https://github.com/evdatsion/inference-anchor).

## Consequences

- Cost is one chain write per batch. Proof size grows with log2 of batch size.
- Proves record integrity and time bounds. Does not prove the model computed the output. That needs TEEs or zk proofs; see [notes/verifiable-ai-stack.md](../notes/verifiable-ai-stack.md).
- The operator keeps raw data and salts. Losing them means receipts can still be shown to exist, but not matched to content. Retention policy has to cover salts explicitly.

## Alternatives considered

- **Signed logs without anchoring.** The operator's signing key can re-sign a rewritten log. Rejected.
- **Full records on chain.** Cost and privacy. Rejected.
- **Third party timestamping (RFC 3161 TSA).** A reasonable fallback where chains are not acceptable, but it moves trust to a single authority.
