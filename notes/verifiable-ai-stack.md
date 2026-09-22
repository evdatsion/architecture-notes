# The verifiable AI stack: which layer do you actually need

"Verifiable AI" covers at least three different guarantees. Mixing them up leads to either overbuilding (zk proofs for a chatbot) or overclaiming (calling a signed log proof of correct computation).

## The three guarantees

| Guarantee | Question it answers | Main tool | Relative cost |
|---|---|---|---|
| Record integrity | Was this output recorded at the time, and left unchanged since? | Hash commitments, Merkle batching, on chain anchors | Very low |
| Execution attestation | Did this output come from the declared model and code, in an environment the operator could not tamper with? | TEEs with remote attestation | Low to moderate |
| Computational proof | Can anyone check, without trusting hardware or the operator, that output = model(input)? | zkML, and optimistic or interactive fraud proofs | High to very high |

## Choosing

**Most enterprise and regulated uses need record integrity first.** Disputes and audits are nearly always about what was said and when, and whether the log was edited. Receipts cover that cheaply. Reference: [inference-anchor](https://github.com/evdatsion/inference-anchor).

**Add execution attestation when the operator is not trusted by the relying party.** Examples: a model provider serving a regulated customer, or several parties sharing one model. GPU confidential computing has made this practical for large models, with the caveat that trust moves to the hardware vendor and to the attestation verification chain.

**Use computational proofs where the model is small and the stakes are high.** Credit scoring, fraud flags, on chain agents that move funds. For large language models, per request zk proving is still far too expensive. Sampling (prove a random subset) or optimistic schemes (prove only when challenged) are the practical middle ground.

## Identity is the part people forget

Every one of these layers needs stable identities: for the model (a digest of weights plus serving config), for the operator (a key that signs receipts or attestations), and often for the end user. Without a clean identity layer, a perfectly valid proof still does not say who is accountable. Treat identity as part of the verifiable stack, not an afterthought.

## Post-quantum horizon

Anchors and attestations are only as durable as the signatures behind them. If audit records need to hold for ten years or more, sign batch roots and attestation reports with a hybrid classical and post-quantum scheme now. Re-signing years of history later is much harder than doing it from the start.
