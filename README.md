# Trade Attestations — Commit-Reveal Ledger

Cryptographic pre-registration of trading decisions from an autonomous
trading research program. Every decision is hashed and published here
**when it is made**; details are revealed **after the outcome exists**.
This makes it mathematically impossible to backfill, cherry-pick, or edit
the track record after the fact.

## How it works
- `commitments.jsonl` — append-only, hash-chained. Each line:
  `{id, sha256, committed_at, source, prev}` where `sha256 =
  SHA-256(salt || canonical_json(decision))` and `prev` is the SHA-256 of
  the previous line (tamper-evident ordering).
- `reveals.jsonl` — after a decision's outcome is known, the full decision
  payload, its salt, and the outcome are published. Recompute the hash and
  match it against the earlier commitment.
- Commit timestamps are anchored by this repository's push history
  (server-side, not author-editable).

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

## What this is not
Not investment advice, not a solicitation, not a performance claim. Paper
and research trades from a system under development; decision *sources*
are labeled but strategy mechanics are intentionally not disclosed.
Everything here is published so that when we do make claims later, this
ledger is the evidence they're checked against.
