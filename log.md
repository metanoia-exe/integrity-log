# Integrity log — tamper-evidence anchors (ADR 0010 §4b)

Each line is the root sha256 over the full provenance record (`indicator_values`
+ `value_sources` + `evidence_snapshots`, canonically serialized and sorted) at
a point in time, as stored in the `integrity_digests` table. This file is
git-tracked and pushed — the commit history is the off-infrastructure anchor
that makes the record tamper-evident: a future auditor can recompute the root
from the open data and check it against a hash that provably existed at commit
time. **History is only provably untampered from the first line below.**

The nightly pipeline writes a digest row every run; this log is appended at
meaningful milestones (methodology bumps, before/after corrections) and
committed. Publishing to a destination outside this repo (public mirror,
transparency log) is a deliberate later step — owner's call on the venue.

Recompute-and-verify: read the three tables via the public API, serialize each
row as the pipe-delimited lines in `lib/evidence.ts` (`runIntegrityDigest`),
sort all lines, sha256 the join. Equal hash = untampered record.

| computed_at (UTC) | values | basket rows | snapshots | root sha256 |
|---|---|---|---|---|
| 2026-07-02T03:15:13Z | 3547 | 720 | 815 | `bbcf3fbc5a3d404c43b1dbe8fc7400383fc5cd9b2e6692c8fba5d05fb37ece72` |
| 2026-07-02T04:01:02Z | 3547 | 720 | 3774 | `6a5c21007b73ded184d4204a909d90d0c9036e23c7a2062a2e6a5925f5668288` |
| 2026-07-04T12:49:51Z | 3678 | 985 | 3855 | 6 | 15 | 5 | `314d9ba394a35ed81c8677cfa674a3d735d40e4dccde6ea1b4f2cf064cdabcbd` |
