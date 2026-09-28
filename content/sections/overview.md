A report never contains the evidence itself. It contains a proof about the evidence.

1. **Blind the target.** The accused identifier (a work email, a platform handle) is turned into an opaque `tag` through an oblivious PRF, so the ledger can't be searched by name.
2. **Commit, don't upload.** The reporter's device publishes a salted commitment to the evidence and a coarse timestamp. The evidence itself stays on the device.
3. **Prove you are one real person.** A zero-knowledge proof shows that the reporter holds a registered credential and that their *nullifier* was derived correctly. The nullifier blocks a second report from the same person against the same target.
4. **Reveal only the count.** Once $k$ distinct nullifiers exist under one tag, a threshold proof tells an intermediary that the threshold has been reached. It reveals nothing else. What happens next depends on each reporter's consent.
