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
| 2026-07-05T10:30:14Z | 3705 | 1046 | 3914 | 6 | 17 | 10 | `a47ea58f0dade224b9634b4a5809ca9f2a2422f9603fca63e3b73e31fd6823f6` |

**Digest v4 — Merkle chain.** Each line commits only the rows appended since
the previous digest: `chain = sha256(prev_chain + "\n" + delta_root)`, seeded
from the last full-scan root above, so every prior line stays inside the
commitment. `lifecycle` is a separate state-at-T root over the mutable ripple
lifecycle tables. Verify by replaying each window's rows from the open data
and folding the chain (`cli digest --verify`).

| computed_at (UTC) | +rows | values | baskets | snapshots | resolutions | ripples | reads | chain root sha256 | lifecycle sha256 |
|---|---|---|---|---|---|---|---|---|---|
| 2026-07-06T06:28:45Z | +21 | 3710 | 1052 | 3921 | 6 | 17 | 10 | `fc8794a1deebfbec3e826b2996aa8031beea9a6b136b61289e811e518d583cbc` | `7b59cd4fae395fbb0a7f0ddf1b8df2ccec194f83fd656e233b375157bb605b62` |
| 2026-07-07T06:07:09Z | +6 | 3710 | 1052 | 3921 | 6 | 19 | 11 | `494c3121f6a98a7de224d84006b0045576a1b8a2ca803d9b7896020835a991c7` | `ec1aba609634d56cdce1f118666ec065f4140d4d60517b60cf34f7acb1a720b8` |
| 2026-07-02T03:15:13Z | full-scan |  | — | — | — | — | — | `bbcf3fbc5a3d404c43b1dbe8fc7400383fc5cd9b2e6692c8fba5d05fb37ece72` | `` |
| 2026-07-02T04:01:02Z | full-scan |  | — | — | — | — | — | `6a5c21007b73ded184d4204a909d90d0c9036e23c7a2062a2e6a5925f5668288` | `` |
| 2026-07-03T06:53:52Z | full-scan |  | — | — | — | — | — | `7adf713b0ac9ef4a4b3423f03e179fb6f7b7b92f3114c15d3463f0a044a00fc9` | `` |
| 2026-07-04T06:27:50Z | full-scan |  | — | — | — | — | — | `4c5daf20bf1e1e0b914b5c2d50029d5d0fb7edf8267c8962f04c06793763feef` | `` |
| 2026-07-04T12:49:51Z | full-scan |  | — | — | — | — | — | `314d9ba394a35ed81c8677cfa674a3d735d40e4dccde6ea1b4f2cf064cdabcbd` | `` |
| 2026-07-05T06:16:12Z | full-scan |  | — | — | — | — | — | `a47ea58f0dade224b9634b4a5809ca9f2a2422f9603fca63e3b73e31fd6823f6` | `` |
| 2026-07-05T10:30:14Z | full-scan |  | — | — | — | — | — | `a47ea58f0dade224b9634b4a5809ca9f2a2422f9603fca63e3b73e31fd6823f6` | `` |
| 2026-07-06T06:28:45Z | +21 | 3710 | 1052 | 3921 | 6 | 17 | 10 | `fc8794a1deebfbec3e826b2996aa8031beea9a6b136b61289e811e518d583cbc` | `7b59cd4fae395fbb0a7f0ddf1b8df2ccec194f83fd656e233b375157bb605b62` |
| 2026-07-07T06:07:09Z | +6 | 3710 | 1052 | 3921 | 6 | 19 | 11 | `494c3121f6a98a7de224d84006b0045576a1b8a2ca803d9b7896020835a991c7` | `ec1aba609634d56cdce1f118666ec065f4140d4d60517b60cf34f7acb1a720b8` |
| 2026-07-07T11:10:31Z | full-scan | 8781 | — | — | — | — | — | `252ebe80b1ac184572598835748e4e921ee79160c89d3c374a8bc58295c6591e` | `ec1aba609634d56cdce1f118666ec065f4140d4d60517b60cf34f7acb1a720b8` |
| 2026-07-08T06:03:03Z | +4 | 3711 | 1052 | 3921 | 6 | 22 | 12 | `353cdbdf58f94b8f6ebb6dc23d15a06b089d227a53ded2fe8f3eb9b0aa60dc15` | `3a5c9d6420359d93bfa50eb00b35211626c659d346709284a004ba2809b191b9` |
