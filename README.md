# PrMaat Audit Anchors

> Public, append-only log of cryptographically-signed Merkle roots from
> the PrMaat audit chain. **GitHub itself is the tamper-evidence layer**
> — every commit hashes the previous, and any rewrite breaks subsequent
> SHAs.

## What's in this repo

For every day PrMaat issues a daily audit-log Merkle root, a signed
canonical-statement file lands here as a separate commit:

- `anchors/YYYY/MM/YYYY-MM-DD.json` — the signed statement artifact
- Commit subject: `anchor YYYY-MM-DD root=<8-hex-prefix>`
- Commit body: short verification recipe

## How to verify an anchor

You don't need to trust prmaat.com. The signed statement is portable.

```bash
# 1. Clone this repo
git clone https://github.com/PrMaat/audit-anchors.git
cd audit-anchors

# 2. Pick a date you want to verify
DATE=2026-05-09
cat anchors/2026/05/$DATE.json

# 3. Reconstruct the canonical statement from (date, root)
python3 -c "
import json, hashlib
a = json.load(open('anchors/2026/05/$DATE.json'))
stmt = f'PRMAAT-AUDIT-ANCHOR-v1\nDATE={a[\"date\"]}\nROOT={a[\"root\"]}'
print('reconstructed sha512 :', hashlib.sha512(stmt.encode()).hexdigest())
print('claimed sha512       :', a['hashedValue'])
print('match                :', hashlib.sha512(stmt.encode()).hexdigest() == a['hashedValue'])
"

# 4. Verify the Ed25519 signature against the platform pubkey
#    (pubkey is embedded in each anchor.json AND served at
#     /.well-known/did.json on prmaat.com)
npx -y @prmaat/verify verify-anchor anchors/2026/05/$DATE.json
```

## Cross-check the chain integrity

The Git commit DAG is the tamper-evidence:

- Each commit references its parent's SHA → rewriting history breaks
  every subsequent commit hash
- GitHub stores commits + mirrors them via clones, watch lists, and the
  GitHub Archive Program
- A rewrite would need to (a) compromise GitHub's git-server-side
  storage, (b) compromise every clone simultaneously — both publicly
  observable

## Spec

[Merkle anchoring protocol — `/spec/v0.2/merkle-anchoring.md`](https://prmaat.com/spec/v0.2/merkle-anchoring.md)

## License

CC-BY-4.0 — anchors are public artifacts. Fork. Verify. Cite.
