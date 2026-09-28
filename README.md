# SafeProof: Privacy-preserving harassment reporting with nullifier-based Sybil resistance and threshold zero-knowledge disclosure.

## Overview
The project is mainly built for the [ACM womENcourage™ 2026 Hackathon](https://womencourage.acm.org/2026/index.php/join-the-hackathon/), with the technical details in[Project page](https://ttgenproject.github.io/safeproof-zkp/), and the pitch version of the hackathon in [Slides](https://docs.google.com/presentation/d/1safyND00YSxTISNe_1uOfcl1YdIe6vnb/edit?usp=sharing)

Reporting harassment means exposing yourself before you know whether anyone else has reported the same person. SafeProof lets a reporter prove two things without revealing their identity or their evidence:

- **A pattern exists:** at least *k* people have reported the same target.
- **They are one distinct, real reporter:** each registered person can report a given target only once.

The intermediary (HR, trust & safety, or an NGO) learns only that the threshold has been reached. Each reporter then decides whether to come forward.

## Protocol

![SafeProof architecture](content/media/architecture.svg)

1. **Blinded target.** `tag = OPRF_k(identifier)`. The ledger never stores the accused's identifier, so it cannot be searched by name.
2. **Evidence commitment.** `c = Commit(evidence, r)`. Evidence stays on the device. The commitment proves the evidence existed by time `t`, not that it is true.
3. **Nullifier.** `nullifier = PRF_s(tag)`. This allows one report per person per target, and the same person's reports against different targets can't be linked. A ZK proof shows registry membership, correct derivation, and binding to `c`.
4. **Threshold proof.** Once `k` distinct nullifiers exist under one tag, an aggregate proof reveals only the count. The design uses either a recursive proof or a `k`-of-`n` escrow.

## Limits

- Sybil resistance is only as strong as the issuer's personhood check.
- A threshold event is a reason to investigate, never a finding. Consent is required at every step.
- The threshold `k` is a policy choice for the deploying institution and its survivor-advocacy partner.
> [!CAUTION]
> SafeProof is a research prototype. It is not a support service, a legal instrument, or a substitute for trauma-informed care.

## Repository

```text
index.html, assets/          static page 
content/site.json            page metadata and links
content/sections/*.md        page text
content/media/               figures
.github/workflows/pages.yml  GitHub Pages deployment
```

Planned components: web intake UI, proof service, ZK circuits, ledger contracts, OPRF service, TypeScript SDK.

**Preview locally:** run `python -m http.server 8000`, then open http://localhost:8000.
**Deploy:** every push to `main` publishes the page. This requires *Settings → Pages → Source: GitHub Actions*.

## References

### Academic references

[A1] Rajan, A., Qin, L., Archer, D. W., Boneh, D., Lepoint, T., Varia, M. "Callisto: A Cryptographic Approach to Detecting Serial Perpetrators of Sexual Misconduct." ACM COMPASS 2018. https://dl.acm.org/doi/10.1145/3209811.3212699

[A2] Hopwood, D., Bowe, S., Hornby, T., Wilcox, N. "Zcash Protocol Specification" — nullifier construction for double-spend prevention under anonymity. https://zips.z.cash/protocol/protocol.pdf

[A3] Semaphore — anonymous signalling with Merkle-tree membership and external nullifiers. https://semaphore.pse.dev/

[A4] Shamir, A. "How to Share a Secret." Communications of the ACM, 22(11), 1979. https://dl.acm.org/doi/10.1145/359168.359176

[A5] Jarecki, S., Kiayias, A., Krawczyk, H. "Round-Optimal Password-Protected Secret Sharing and T-PAKE in the Password-Only Model" — OPRF constructions underlying blinded identifier derivation. IACR ePrint 2014/650. https://eprint.iacr.org/2014/650

[A6] Gabizon, A., Williamson, Z. J., Ciobotaru, O. "PLONK: Permutations over Lagrange-bases for Oecumenical Noninteractive arguments of Knowledge." IACR ePrint 2019/953. https://eprint.iacr.org/2019/953

[A7] Groth, J. "On the Size of Pairing-Based Non-interactive Arguments." EUROCRYPT 2016. https://eprint.iacr.org/2016/260

### Code inspirations

[C1] Semaphore protocol — reference implementation of the membership-plus-nullifier circuit pattern. https://github.com/semaphore-protocol/semaphore

[C2] Tornado Cash circuits — Merkle membership and nullifier gadgets in circom. https://github.com/tornadocash/tornado-core
