# BFT in production: what breaks first

Notes from building and operating chains on Tendermint style BFT consensus with Cosmos SDK derived application layers. The protocol is well understood. The problems are almost always around it.

## Networking before consensus

Most "consensus failures" we chased were really peer problems. Validators behind NAT with bad external address config, sentry nodes that were not actually shielding anything, persistent peer lists that pointed at decommissioned hosts. Before tuning timeouts, check that every validator can reach at least a third of voting power directly through its sentries.

## Timeouts are a product decision

Propose and commit timeouts set the block time, and block time is something users feel. Shorter timeouts give faster blocks, until validators on the far side of the planet start missing rounds. Measure the real round trip distribution across the validator set and set timeouts from its tail, not from the median.

## State growth is the slow killer

IAVL based state works well at launch and gets slower as the tree grows. Pruning strategy, snapshot intervals and state sync need to be designed before mainnet, because changing them afterwards means every operator does a painful migration at the same time. Budget for disk growth that looks unreasonable on day one.

## Upgrades

Coordinated upgrades at a fixed height work well when the release is boring. They fail when:

- binaries are not reproducible, so validators run slightly different builds,
- store migrations are not tested against a copy of real mainnet state,
- the halt height leaves operators no time zone friendly window.

Tooling like cosmovisor helps, but the real fix is a dry run on exported mainnet state for every upgrade.

## Key management

Double signing is far more often an operations mistake than an attack: two instances of the same validator started after a failover. Remote signers with double sign protection, and a failover runbook that someone has actually rehearsed, prevent most slashing incidents.

## Light clients and bridges

Every change to validator set rules, header format or signature scheme has to be checked against every light client and bridge that verifies your headers. Keep a list of them. It is always longer than people expect.
