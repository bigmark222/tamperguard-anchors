# tamperguard-anchors

Append-only log of the two out-of-band anchors `tamperguard` prints after each promote.

**This repo contains hashes and nothing else.** A SHA-256 of a script and a SHA-256 of a
file-manifest reveal nothing about their contents. It is public on purpose.

## Why public

Branch protection on private repositories requires a paid GitHub plan; on public ones it
is free. Public therefore buys **force-push blocked, deletions blocked, and required
signed commits** at no cost — which is the entire point. An attacker holding the machine
*and* the medium still cannot rewrite this history; they could at most append to it, and
appends are visible.

The cost is metadata: this reveals that the machine runs tamper detection, and how often
it is promoted. That is an acceptable trade for an append-only witness, and secrecy about
whether you have tripwires is not a security property worth paying for.

## Why it exists

The anchors are the root of tamperguard's trust model. `verify.sh` cannot vouch for
itself, and the on-medium `manifest.sha256` cannot vouch for itself either — both are
checked against a value you hold **out of band**. Until this repo, that value lived in
exactly one place: a password manager on the same machine being audited.

Now it lives in three: password manager, paper, and here — where it is append-only and
off-machine.

## The first row is a TEST — do not match it against the medium

The row noted `simulated bootstrap` (2026-07-30) was produced while building
`anchor-publish.sh`, against a **simulated** medium in a scratch directory — not against
real hardware. Its anchors correspond to nothing you own.

It is still here because this log is append-only and force-push-protected, which is the
whole point: **not even I can quietly remove an inconvenient row.** That is the property
being demonstrated, so the row stays and this note explains it.

The first genuine row will carry the note `bootstrap` and will follow the first real
promote to physical media.

## Format

`anchors.tsv`, one row per promote, tab-separated:

```
timestamp<TAB>script_anchor<TAB>manifest_anchor<TAB>note
```

- **script_anchor** = `sha256(verify.sh)` — changes only when `verify.sh` changes.
- **manifest_anchor** = `sha256(manifest.sha256)` — changes on every promote.

## Verifying against it

```sh
shasum -a 256 /Volumes/TAMPERGUARD/verify.sh          # compare to the last script_anchor
shasum -a 256 /Volumes/TAMPERGUARD/manifest.sha256    # compare to the last manifest_anchor
tail -1 anchors.tsv
```

A mismatch you cannot account for means the medium or the checker changed. See
`docs/06-recovery.md` in the tamperguard repo.

## Third-party timestamping

Each append is stamped with [OpenTimestamps](https://opentimestamps.org), producing
`anchors.tsv.ots`. That anchors the log's hash into the Bitcoin blockchain via free
public calendars, so **retroactive** rewriting is detectable even if this repo, the
machine, the medium and the GitHub account were all compromised. Proofs are pending for
a few hours after stamping; `ots upgrade anchors.tsv.ots` completes them, and
`ots verify anchors.tsv.ots` checks one offline afterwards.
