This section answers the questions we expect from the jury. You do not need a background in cryptography to read it. Each answer starts with the plain-language version and then gives the technical term in brackets, so you can match it to the Protocol section.

### The idea in plain words

#### 1. What does SafeProof do, in one sentence?

SafeProof lets a person report harassment and prove *"I am one real person, and I am not the only one who reported this individual"* without revealing their name, their evidence, or even the accused's name to the public record.

Nobody is identified until enough independent people have reported the same person, and even then, each reporter decides for themselves whether to step forward.

#### 2. Why is this needed? Can't people just report to HR or to the platform?

They can, but in practice many do not. The Why Reporting Fails section lists the reasons. In short:

- **The reporter pays first.** You have to reveal who you are before anyone has decided anything, so you carry the retaliation risk alone.
- **"It's just one person's word."** A single report is easy to dismiss. The only way to find out whether others have reported the same person is to expose yourself.
- **Telling the story again and again.** Every new platform or investigator asks for the full account from the start.
- **Patterns stay invisible.** The same person harassing at work and online shows up as unrelated incidents.

SafeProof changes the order: **be counted first, be identified later, and only if you choose.**

#### 3. Who uses it?

| Who | What they do |
|---|---|
| **Reporter** | A person who experienced harassment. Files a report from their own device. |
| **Identity issuer** | Checks once that each reporter is a real, unique person (for example, a university or employer account check). Does not see any reports. |
| **Intermediary** | HR, a trust & safety team or an NGO. Is told only *"at least $k$ distinct people reported this person"*. |
| **Institution + survivor-advocacy partner** | Decide the threshold $k$ and the rules for what happens next. |

### Zero-knowledge proofs

#### 4. What is a zero-knowledge proof?

A zero-knowledge proof lets you **convince someone that a statement is true without showing them why it is true**.

An everyday example: at a bar, you need to prove you are over 18. Today you hand over your ID card, which also shows your full name, your address and your exact birth date. With a zero-knowledge proof, you could prove *"I am over 18"* and the bartender would learn **that one fact and nothing else**, while being mathematically certain it is true.

A classic illustration is the *Where's Waldo?* puzzle. You want to prove you found Waldo without showing where he is. You put a large sheet of cardboard with a small hole over the page so that only Waldo shows through the hole. Your friend sees Waldo, so they know you found him, but they have no idea where on the page he is.

A zero-knowledge proof has three properties:

1. **Honest proofs are always accepted** *(completeness)*.
2. **A false claim cannot pass**, except with a chance so small it can be ignored (smaller than guessing a random 70-digit number) *(soundness)*.
3. **The checker learns nothing except that the claim is true** *(zero knowledge)*.

#### 5. What exactly does a reporter prove in SafeProof?

With each report, the reporter's device produces a proof of three statements:

1. **"I am a registered, real person."** I hold one of the membership passes that the identity issuer handed out, but I won't tell you which one *(Merkle-tree membership)*.
2. **"I have not already reported this person."** My report carries a one-time stamp that is unique to me *and* to this accused person. A second report from me against the same person would carry the same stamp and would be rejected *(nullifier)*.
3. **"My evidence existed at this time."** My evidence is sealed in a digital envelope and the seal is recorded with a coarse date. The evidence itself never leaves my phone *(commitment)*.

When $k$ reports have been collected, one more proof tells the intermediary: *"At least $k$ **different** real people have reported this person."* The intermediary learns the count, not who reported and not what they reported *(threshold proof)*.

#### 6. And what does it *not* prove?

We want to be clear about this, because it matters for fairness:

- It does **not** prove that the harassment happened, or that the evidence is true.
- It does **not** prove that the reporter ever met the accused.

It proves that **$k$ distinct real people each filed a report**, and that each piece of evidence existed by a certain date. This is a **reason to investigate, never a verdict**.

#### 7. What is a *non-interactive* zero-knowledge proof, and why does it matter here?

The first zero-knowledge proofs were **interactive**, like an oral exam. The checker asks a random question, the prover answers, and they repeat many rounds until the checker is convinced. Both have to be online at the same time, and the conviction only holds for that one checker, who saw the questions being random.

A **non-interactive** proof is more like a **sealed, signed certificate**. The prover creates it once, alone, on their own device. Anyone can check it later, any number of times, without talking to the prover. Technically, the random questions are replaced by a fingerprint of the statement itself, so the prover cannot choose them to their advantage *(the Fiat–Shamir transform)*.

For SafeProof this is essential:

- **The reporter submits once and walks away.** There is no live conversation with a server that could log their network address or timing.
- **Anyone can re-check the record at any time.** An auditor, a court or the accused's representative can verify every report proof without contacting any reporter.
- **The shared ledger can check proofs automatically.** A program on the ledger (a smart contract) cannot hold a conversation, but it can check a certificate.

The family of proofs we build on is called **zk-SNARK**. The "N" stands for *Non-interactive*, and the "S" for *Succinct*: the proof is a few hundred bytes and takes milliseconds to check, however complex the statement.

#### 8. zk-SNARK or zk-STARK: which one, and what is the trade-off?

Both are non-interactive zero-knowledge proofs. They differ in their trade-offs:

| | **zk-SNARK** (e.g. Groth16, PLONK) | **zk-STARK** |
|---|---|---|
| Proof size | Very small (hundreds of bytes) | Larger (tens to hundreds of kilobytes) |
| Cost to verify on a ledger | Low | Higher |
| One-time setup ceremony | Needed (*trusted setup*) | Not needed (*transparent*) |
| Resistant to future quantum computers | No | Yes, believed to be |

The **trusted setup** is a one-time ceremony that creates public parameters. If *every* participant in the ceremony cheated and kept their secret, they could forge proofs. In practice, ceremonies such as the ones used by Zcash and Ethereum projects involve many independent participants, and only **one** of them needs to be honest.

For a hackathon prototype, we lean towards zk-SNARKs, because they are small and cheap to verify, and the tooling is mature: the Semaphore and Tornado Cash circuits we build on are written in Circom. Comparing this with a STARK-based version is part of our future work.

### Blockchain

#### 9. Why use a blockchain at all? Why not a normal database run by HR?

The system is only trustworthy if **no single organisation can quietly change the record**. With a normal database, whoever runs it can:

- delete reports (for example, to protect a powerful employee),
- back-date or edit entries,
- claim the threshold was never reached,
- or add fake reports.

A blockchain is a **shared notebook that many independent parties keep a copy of**. New entries are added only when the parties agree *(consensus)*, and old entries cannot be changed without everyone noticing *(tamper-evidence)*. So:

- **The institution being reported to does not control the record.** This is important when the accused may be senior staff.
- **The rules are public code.** The program that accepts reports, rejects duplicates and counts toward $k$ is published and runs the same way for everyone *(smart contract)*.
- **Anyone can audit it.** The count of reports against a blinded tag, and every proof, can be checked independently.

#### 10. Does that mean personal data goes on a public blockchain forever?

**No, and this is a deliberate design choice.** The ledger stores only:

- the **blinded tag** of the accused (a scrambled code, not their name or email),
- **one-time stamps** (nullifiers), which cannot be traced back to a person,
- **sealed envelopes** of the evidence (commitments), which reveal nothing about the content,
- **coarse timestamps**.

Names, evidence, messages and screenshots **stay on the reporter's device**. Because nothing personal is written to the ledger, the fact that the ledger is permanent does not conflict with privacy rights such as the GDPR's right to be forgotten. The reporter can delete their own evidence at any time.

#### 11. Which blockchain will you use?

We have not fixed this yet, and the design does not depend on a particular chain. It needs a ledger that can run a small program to check zero-knowledge proofs. That could be a public network, such as Ethereum or a cheaper layer-2 network on top of it, or a **permissioned ledger** run jointly by several institutions and NGOs. A public network gives the strongest independence. A permissioned one gives lower cost and more control over who operates it. The choice is a deployment decision.

#### 12. Is there a cryptocurrency involved? Does the reporter need a crypto wallet?

The reporter should never have to handle cryptocurrency. In the planned design, a proof service (a *relayer*) submits the report on the reporter's behalf and pays any network fee. This also stops a payment from linking the report to a wallet.

### Identity and privacy

#### 13. How do you know each report comes from a different, real person? Couldn't someone create 100 fake accounts?

This is known as a **Sybil attack**, and preventing it is central to SafeProof.

1. **One pass per person.** An identity issuer (for example, the conference registration desk or a university account system) checks *once* that you are a real, unique person and registers one anonymous membership pass for you. It never sees your reports.
2. **One stamp per person per accused.** When you report, your pass and the accused's tag are combined into a one-time stamp *(nullifier)*. The same person reporting the same accused twice always produces the same stamp, so the ledger rejects the second report.
3. **Different accused, unlinkable stamps.** Reports you make against *different* people produce completely unrelated stamps, so nobody can build a profile of everything you reported.

A good comparison is an election. You can vote only once, and everyone can check that no one voted twice, but the ballot does not carry your name.

#### 14. How is the accused's name hidden? Couldn't someone just try every email address?

If we simply scrambled the email address with a standard formula *(hash)*, an attacker could scramble every address in the company directory and compare the results.

To prevent this, the scrambling is done **jointly with a separate key-holding service**. The service adds a secret ingredient without ever seeing the email address, and the reporter's device never learns the secret ingredient *(oblivious pseudo-random function, OPRF)*. Nobody can scramble addresses offline, so a dictionary attack does not work. Callisto, an existing sexual-assault reporting system, uses the same defence.

#### 15. Who learns what?

| Party | Learns | Never learns |
|---|---|---|
| Public / ledger | That a valid report was filed against a blinded tag, and when (roughly) | Reporter's identity, evidence, accused's name |
| Identity issuer | That you are a registered person | Whether, when or whom you reported |
| Key service (OPRF) | That some tag was computed | Which person the tag refers to |
| Intermediary, before $k$ | Nothing | Anything |
| Intermediary, after $k$ | "At least $k$ distinct people reported this person" | Who they are, unless each one consents |

#### 16. What happens once the threshold $k$ is reached?

The intermediary receives a notification that the threshold was reached. It can then send an **invitation** through a private channel to the reporters. **Each reporter decides individually** whether to come forward, stay anonymous or withdraw. Consent is asked at every step and never carries over:

> Filing a report is not consent to escalation. Escalation is not consent to identification. Being identified to one intermediary is not consent to being identified to another.

### Challenges and hard questions

#### 17. Couldn't a group of people coordinate false reports?

Yes, and no technology can prevent people from lying. What SafeProof does:

- It makes coordinated abuse **costly**: each attacker needs a real, separately verified identity, and can report each person only once.
- The threshold $k$ is set by the institution together with a survivor-advocacy partner, to balance *"victims not being heard"* against *"unfounded investigations"*.
- A threshold event is **a reason to investigate, never a finding**. The normal process, with due process for the accused, still applies.

#### 18. Isn't all the trust now just placed in the identity issuer?

Partly, yes, and we say so openly: **Sybil resistance is only as strong as the issuer's personhood check.** If the issuer lets one person register twice, that person can report twice. What the issuer *cannot* do is see or link reports, because the issuer only knows who holds a pass, not what the passes were used for. Using a trusted issuer that already exists (for example, a university or an employer's HR system) keeps the design realistic.

#### 19. Isn't this too slow or too heavy for a phone?

The per-report proof covers a small statement (membership, one stamp and one envelope). Proofs of this size, such as Semaphore proofs, are typically generated in seconds in a phone or laptop browser. Checking a proof takes milliseconds. The heaviest part is the **threshold proof** over $k$ reports. For this we propose two options:

- a **recursive proof** (a proof about many proofs), where you trust only the mathematics but it costs more computation, or
- a **key-splitting escrow**, where the key is split into $k$ pieces and $k$ pieces are needed to unlock it *(Shamir secret sharing)*. This is lightweight, but you must trust the escrow operator.

Choosing between them is one of the trade-offs we want to evaluate.

#### 20. What about laws that force HR to act once they are told?

In many jurisdictions, an intermediary *must* act once notified of certain conduct *(mandatory reporting)*. A threshold notification could therefore start a process the reporters did not choose. SafeProof must explain this in plain language **before** anyone files, and every real deployment needs legal review in its own jurisdiction.

#### 21. Is it safe to use with real survivors today?

**Not yet.** SafeProof is a research prototype. It is not a support service, a legal instrument, or a substitute for trauma-informed care. Before any use with real reporters, it needs institutional ethics review, legal counsel and a survivor-advocacy partner, and every deployment must direct reporters to human support.

#### 22. What does the next step look like?

The building blocks are defined in the Why womENcourage section:

- the report and threshold circuits,
- the ledger contracts (registry, commitments, nullifier set, escrow),
- the blinded-identifier service,
- a web intake app and an intermediary console.

Next, we will build a working end-to-end prototype, measure proof times and costs, compare zk-SNARK and zk-STARK variants, and run a design review with survivor-advocacy organisations.

### One-line answers

For quick replies during the live Q&A:

| If the jury asks… | One-line answer |
|---|---|
| *What is a zero-knowledge proof?* | Proving something is true without showing why, like proving you're over 18 without showing your ID. |
| *Why non-interactive?* | The reporter creates one self-contained proof, submits it and leaves. Anyone can check it later without contacting them. |
| *Why blockchain?* | So that no single organisation, including the one being complained to, can delete, edit or hide reports. |
| *Is personal data on-chain?* | No. Only scrambled tags, one-time stamps and sealed envelopes. Evidence stays on the phone. |
| *How do you stop fake accounts?* | One verified pass per person and one stamp per person per accused. Duplicates are rejected automatically. |
| *Does it prove the harassment happened?* | No. It proves $k$ distinct real people reported. That is a reason to investigate, not a verdict. |
| *Who decides what happens next?* | Each reporter, individually, at every step. |
