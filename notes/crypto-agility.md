# Crypto agility in transaction and key formats

"Crypto agility" gets used to mean "we could swap algorithms if we had to." On a chain, that is only true if the formats were designed for it on day one. Most were not, which is why algorithm changes usually end up as hard forks.

## What actually has to be agile

1. **Key encoding.** Every public key carries an algorithm identifier. A one byte suite prefix is enough, and it is much cheaper than guessing by length.
2. **Signature encoding.** Same prefix, and verifiers reject a signature whose suite does not match the key's suite before doing any math.
3. **Address derivation.** Hash the full encoded key, suite byte included. Then two suites can never produce the same address from related key material.
4. **Account state.** Store a key hash and a suite, not a raw key in a fixed size field. Fixed size fields are where agility goes to die.
5. **Fee and size limits.** Parameterized per suite. A 3 KB signature and a 64 byte signature should not cost the same to include.
6. **Verification dispatch.** One registry that maps suite to verifier, with an activation height for each entry. Adding a suite is a registry change behind a governance upgrade, not code spread across the state machine.

## What does not need to be agile

Hash functions used for Merkle trees and state commitments. SHA-256 and SHA3-256 hold up against known quantum attacks at the security levels chains care about. Making them swappable adds a lot of complexity for no real gain. Pick one, document it, move on.

## Things that go wrong

- **Suite confusion.** A verifier that accepts any suite for a key lets an attacker choose the weakest one. The suite is bound to the account, not chosen per transaction.
- **Silent downgrade during migration.** If both legacy and new suites are accepted in the overlap window, an account that has rotated must stop accepting legacy signatures right away. Otherwise rotation protects nothing.
- **Unbounded suite list.** Every suite is attack surface and maintenance. Retire suites on a schedule, the same way they were added.
- **Hardware wallets.** Agility on chain does nothing if the signer cannot run the new algorithm. Check device constraints before choosing the suite, not after.
