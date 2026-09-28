The womENcourage™ 2026 theme is *"Unmute Yourself, Grow Stronger Together."* SafeProof is a cryptographic reading of both halves:

- **Unmute yourself** without being exposed. A reporter can be counted before they can be identified, so speaking up no longer requires paying the full price of visibility up front.
- **Grow stronger together.** One report is easy to dismiss. $k$ independent reports are a pattern. The threshold proof lets reports gain weight *together*, and still lets each reporter decide for themselves whether and when to come forward.

The hackathon asks how computing can drive well-being and support for women in computing. Harassment at work, at conferences and on platforms pushes people out of computing, and much of it is never reported. SafeProof targets the structural barrier that keeps that harm unreported.

#### Planned components

| Component | Role |
|---|---|
| `apps/web` | Reporter intake UI and intermediary console (HR / T&S / NGO) |
| `apps/prover` | Proof service: report proofs and threshold aggregation |
| `packages/zk-circuits` | Report circuit, aggregation circuit, Merkle and nullifier gadgets |
| `packages/contracts` | Ledger logic: registry, commitments, nullifier set, escrow |
| `packages/identifier-oprf` | Blinded target-identifier derivation (OPRF service and client) |
| `packages/safeproof-sdk` | Shared TypeScript SDK: commitments, proof orchestration, transaction builders |
