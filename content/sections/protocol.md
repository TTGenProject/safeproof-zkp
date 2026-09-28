### Blinded target identifiers

A report is filed against a **target identifier**: something that persistently and uniquely points to the accused, never to the reporter. Committing $H(\text{id})$ directly would be catastrophic. The space of identifiers is small enough to enumerate, so anyone could run a dictionary attack on the ledger and learn who was reported and how often. SafeProof derives the ledger key through an **oblivious pseudorandom function** instead:

$$\text{tag} = \mathrm{OPRF}_k(\text{id})$$

The client and a key-holding service evaluate it together. The service never sees the identifier and cannot compute tags offline. Callisto uses the same defence for perpetrator identifiers [A1]. Without it, the reporter would stay anonymous while the *accused* faced unaccountable public accusation, which is a different but equally serious failure.

### Anonymous evidence commitment

The client computes $c = \mathrm{Commit}(\text{evidence} \,\|\, \text{narrative};\ r)$ over material held only on the device: a screenshot, an email, a written account. It submits $(\text{tag}, c, t)$, where $t$ is a coarse timestamp.

This does **not** prove that the evidence is genuine or that the account is true. It proves that *this exact material existed no later than $t$*. That rules out the two most common procedural attacks in harassment disputes: disputing the timeline after the fact, and claiming the report was made up after some later event.

### Nullifier-based Sybil resistance

Each reporter holds a secret $s$ and a leaf $\mathrm{Commit}(s)$ in a registry Merkle tree. An issuer admits the leaf after checking personhood, without learning what the credential is later used for. For a report against a given tag, the client derives

$$\text{nullifier} = \mathrm{PRF}_s(\text{tag})$$

and the ledger rejects any nullifier it has already seen. This is the construction used by Semaphore and Zcash [A2, A3]. The nullifier is deterministic in $(s, \text{tag})$, so one person can file **at most one report per target**. It is also pseudorandom across targets, so reports by the same person against different targets **cannot be linked**.

The circuit proves in zero knowledge that (i) $\mathrm{Commit}(s)$ is in the registry at a known root, (ii) the nullifier was derived correctly from the same $s$, and (iii) the commitment $c$ is bound to this report.

> **Scope of the guarantee.** Sybil resistance here means exactly one report per registered person per target, and nothing more. It does not show that the reporter ever had contact with the accused, and it inherits every weakness of the issuer's personhood check.

### Threshold proof and escalation

Nothing is published when a report is filed. Nothing is published when the threshold is crossed either. Once $k$ distinct nullifiers exist under one tag, an aggregation proof states:

> *There exist $k$ pairwise-distinct nullifiers under this tag, each derived from a distinct registered credential, each bound to a commitment recorded at or before its stated time.*

| Mechanism | How it works | Where the trust sits |
|---|---|---|
| **Recursive proof over the nullifier set** | $k$ report proofs are folded into one recursive proof of cardinality. | Only the proof system. The intermediary learns $k$ and nothing else. |
| **$k$-of-$n$ matching escrow** | Each report carries a share of a key that encrypts the reporter's contact channel. The $k$-th share makes reconstruction possible [A1, A4]. | How the escrow operator handles keys, and the assumption that fewer than $k$ shares leak nothing. |

### Cross-context linking

The tag comes from an identifier, not from a context. That means reports filed from a work Slack, a conference badge, an X account and a Discord server can be shown to concern one person, but only once the identifiers are known to belong to the same person, and that is not a cryptographic fact. This leaves two designs:

- **Reporter-asserted linking.** The reporter links the identifiers themselves. It is cheap and needs no new trusted party, but it is wrong whenever the reporter is mistaken or acting in bad faith.
- **Attested linking.** An issuer or platform confirms that the identifiers belong to the same person. This is sound, but it brings back a party that learns which identifiers are being linked.
