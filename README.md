# Trade Attestations, Commit-Reveal Ledger

Cryptographic pre-registration of trading decisions from an autonomous
trading research program. Every decision is hashed and published here
**when it is made**; details are revealed **after the outcome exists**.

---

## READ THIS FIRST: the first 40 commitments are NOT time-anchored

This repository was created on 2026-07-21 and was supposed to receive a
push every day. **It never received one.** The daily job's `git push` failed
silently every single time with `The current branch master has no upstream
branch`, and nobody noticed until 2026-08-07. The remote had no branches at
all until that date.

So the first publication of this repository, on **2026-08-07**, contains
**40 commitments dated 2026-07-21 through 2026-08-06 that were written
locally and published late.**

**What that means, stated plainly:**

- For those 40 records, **the GitHub timestamp proves nothing.** It says
  2026-08-07, which is after the outcome of nearly every decision in them.
  A hash written on your own disk is not evidence of anything, because
  anyone can write a file and claim a date.
- We are **not** asking anyone to treat those 40 as pre-registered. They
  are published for completeness and because deleting them would be worse.
  Treat them as **UNANCHORED**.
- The internal hash chain across them is intact and verifiable (each line
  carries the SHA-256 of the previous line), and the 6 reveals recompute
  exactly. That proves the file has not been edited since it was written.
  It does **not** prove when it was written. Those are different claims and
  only the second one matters for pre-registration.

**The boundary.** Everything up to and including this record is unanchored:

```
id      swing:NVDA:2026-08-06T13:20:24Z
sha256  b110b54ebc6c9b8c9955f50df8feaf9bea089c80f9bc0736374ba3214238e5bb
```

Every commitment appearing **after** that line arrived in its own push, on
or after 2026-08-08, and its GitHub push timestamp is real third-party
evidence of when it existed. The anchored record starts there.

**Why we are telling you this.** The entire point of a commit-reveal ledger
is that you do not have to trust us. Quietly pushing 40 backdated-looking
hashes and letting the repository imply they were published on time would
have produced exactly the kind of misleading record this program exists to
detect in others. The failure was ours, it was caught by our own review, and
this notice is the correction.

---

## How it works

- `commitments.jsonl`, append-only and hash-chained. Each line:
  `{id, sha256, committed_at, source, prev}` where `sha256 =
  SHA-256(salt || canonical_json(decision))` and `prev` is the SHA-256 of
  the previous line (tamper-evident ordering).
- `reveals.jsonl`, published after a decision's outcome is known: the full
  decision payload, its salt, and the outcome. Recompute the hash and match
  it against the earlier commitment.
- Commit timestamps are anchored by this repository's push history
  (server-side, not author-editable), **for records after the boundary
  above only**. The `committed_at` field is self-reported and is not
  independent evidence on its own.

## Verify it yourself

```python
import hashlib, json

def check(reveal_line, commitments):
    r = json.loads(reveal_line)
    payload = json.dumps(r["payload"], sort_keys=True, separators=(",", ":"))
    h = hashlib.sha256((r["salt"] + "|" + payload).encode()).hexdigest()
    assert h == commitments[r["id"]], "MISMATCH"
    return r["id"], "verified"

commits = {json.loads(l)["id"]: json.loads(l)["sha256"]
           for l in open("commitments.jsonl")}
for line in open("reveals.jsonl"):
    print(check(line, commits))
```

To check the chain itself, confirm that each line's `prev` equals the
SHA-256 of the entire previous line, and that the first line's `prev` is
`genesis`. To check *when* a record existed, use this repository's commit
history, not the `committed_at` field.

## What this is not

Not investment advice, not a solicitation, not a performance claim. Paper
and research trades from a system under development; decision *sources* are
labeled but strategy mechanics are intentionally not disclosed. Everything
here is published so that when we do make claims later, this ledger is the
evidence they are checked against.
