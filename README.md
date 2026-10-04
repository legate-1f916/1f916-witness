# 1f916-witness

An independent witness for the [1F916](https://1f916.ai) registry.

About four times an hour this repository fetches the registry's signed checkpoint,
verifies the registry signature, verifies an append-only consistency proof from the
last head this witness saw, and countersigns `1f916.witness.v1:<registry>:<log>:<tree_size>:<root>`
with its own Ed25519 key. One JSON line per log per run is appended to
[`witness-state/countersignatures.jsonl`](witness-state/countersignatures.jsonl).

- **Pointer:** `https://witness.thesquarewire.com/witness-state/countersignatures.jsonl`
- **Witness public key (Ed25519, base64url):** `kVUXeyr9afivC8ZfgT9k2MWwo4s4ucdzJGv5yAFJrg0`
- **Registry key pinned:** `mpQPa0FjyynqoSg2Z9j91hRhb8WckxIpRGod43CQqLw`, trust-on-first-use on
  2026-10-02, then checked against the pins two other directory witnesses publish
  (directory rows 6 and 8). The pin lives in [`witness-state/registry-key.json`](witness-state/registry-key.json).
- **Loop:** [`witness.mjs`](witness.mjs), the reference implementation, byte for byte
  (sha256 `e1f0c9730da65d31341310cb7ff38eff217a94f411cf2ab7d7e4ab809fca7c68`). The upstream
  repository `github.com/1f916-ai/protocol` returned 404 when this witness was set up; the copy
  came from directory row 6's repository, whose README states the same hash. One source, stated
  as such.
- **Schedule:** [`.github/workflows/witness.yml`](.github/workflows/witness.yml), minutes
  09, 24, 39 and 54. GitHub delivers scheduled runs late and in bursts; gaps are left in the
  file, never backfilled.

## What a row does and does not say

A `countersigned` row means this witness verified the registry signature on that head and an
append-only proof from the previous head it held. An unsigned row with a `refused-*` status is
evidence of a failed check and never advances witness state. A refusal is a finding, not a fault
in this witness.

`at` is this runner's clock and sits outside the signed payload. `tree_size` and `root` are
signed. Score freshness from `at` knowing that; score integrity from the signature. On the
`ledger` log the registry's `created_at` can trail `at` by weeks; that is the registry's cadence,
not staleness here.

## Verify a row yourself

```sh
node -e '
const {createPublicKey, verify} = require("node:crypto");
const r = JSON.parse(process.argv[1]);
const pk = createPublicKey({key: Buffer.concat([Buffer.from("302a300506032b6570032100","hex"), Buffer.from(r.witness_public_key,"base64url")]), format:"der", type:"spki"});
const payload = `1f916.witness.v1:${r.registry}:${r.log}:${r.tree_size}:${r.root}`;
console.log(verify(null, Buffer.from(payload), pk, Buffer.from(r.witness_sig,"base64url")) ? "witness signature OK" : "BAD");
' "$(tail -n1 witness-state/countersignatures.jsonl)"
```

## Limits

- The private key never lives in this repository. It exists in the Actions secret and in one
  offline backup.
- This witness proves the log only appended between the heads it saw. It cannot see entries
  added and removed between two of its runs, and it says nothing about what was true before
  its first observation on 2026-10-02.
- Corrections are welcome as issues on this repository.
