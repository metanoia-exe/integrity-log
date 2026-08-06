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
| 2026-07-09T06:07:05Z | +4 | 3711 | 1052 | 3921 | 6 | 23 | 13 | `996394c1f001862db5f75fe5380f53a3721323fe06dc133dcb8cd2de95d9a327` | `51930cd59100f3daa8e927b926efd9ae139be5c504f475e03e292ed1a76e3871` |
| 2026-07-10T06:02:56Z | +4 | 3712 | 1053 | 3921 | 7 | 23 | 14 | `54dcdfe7c5098e8ead3d2455879e39225b6f15885a7ed38f31c25db000198164` | `51930cd59100f3daa8e927b926efd9ae139be5c504f475e03e292ed1a76e3871` |
| 2026-07-11T06:04:53Z | +6 | 3712 | 1053 | 3921 | 7 | 26 | 15 | `608fc9f47f84d159f29acd92a5575fdb3f5e3e6389850deb07707aed061436f0` | `43d1b43dad6530cbc74c48739a63bfdcff98746411c12d496f186eddf022b158` |
| 2026-07-12T06:04:45Z | +1 | 3712 | 1053 | 3921 | 7 | 26 | 16 | `861f1d09ed93b2c1cf5ae6690835fb9d3caae1d949728768f1c52bfa85cf3160` | `43d1b43dad6530cbc74c48739a63bfdcff98746411c12d496f186eddf022b158` |
| 2026-07-13T06:28:27Z | +10 | 3713 | 1055 | 3923 | 7 | 28 | 17 | `738db7f443153315831dbd36efb9e35bb0e5a7e83b5b96492f86aa8357b70521` | `a8af57759cddd9cd80b2bc9975c369995cc56615d8d1cbe87dce32f28e3f7234` |
| 2026-07-14T06:05:14Z | +5 | 3713 | 1055 | 3923 | 7 | 30 | 18 | `7c3e376f42b3f5d5231bd499e064414c92e9069010dcaff6e46a2cb7adb5efab` | `92e43e5811b5a910e9edcef8311390177e5b111c3c43beda5358ad2dd7a73406` |
| 2026-07-15T06:05:46Z | +5 | 3713 | 1055 | 3923 | 7 | 33 | 19 | `f18e64dfe7aba17a57846f681aaa9df0a8cfa5b5bcc32adb5dd693f89d76237f` | `e8ce187cc325f6121484b368294ac688f0596aa122510eff0c54cc118376980b` |
| 2026-07-16T06:02:45Z | +4 | 3713 | 1055 | 3923 | 7 | 34 | 20 | `6d6c36cf10072c9246795f9a342cc59bad258ad78638521dcfca2d46f5d3a0b7` | `1ae631dcdbbf6899dd4a3231e4a3848e55b67a31948a44d7c15231e09d715280` |
| 2026-07-17T06:05:23Z | +1 | 3713 | 1055 | 3923 | 7 | 35 | 21 | `be634cdc9fa258dbf0356030cd88677740dab8c2e37b1e26e74fedbb5fe69d94` | `08052c34a60b62ff409fbb6013d14d3ee089e2e77bbaa916406db0c79dcbb340` |
| 2026-07-18T06:04:41Z | +1 | 3713 | 1055 | 3923 | 7 | 36 | 22 | `967cb911a7e1c8dfda2f722800167eb39588cea5ddc3fd290e1f44fc274c28b6` | `cb8c2f06f3c049252d7b4e554ab1f03042e4a2fe0d753b9371feb8aa22244b05` |
| 2026-07-19T06:10:11Z | +2 | 3713 | 1055 | 3923 | 7 | 38 | 23 | `42a799a33690cd70d54fbcf285b619f075fc7ff9d38a1a754da1749ead6d2848` | `101dbf1110d3e064145fb1e17f878433812377db1189037b7814132ee3a8638d` |
| 2026-07-20T06:12:00Z | +15 | 3715 | 1060 | 3928 | 7 | 42 | 24 | `0b53b2658ab51044c325f93907476e394f12c4fa3a1cba8d9ea9c0fa9b398ea5` | `9da8b2fc3ccb2f1d9fba920a63a2d95fbefccda04d5c54e2849638377418ea3b` |
| 2026-07-21T06:10:23Z | +1 | 3715 | 1060 | 3928 | 7 | 43 | 25 | `e6c7273b59c2d0956880136b70f1a8924a575cec80b6dfd3310a1f0705b348e2` | `e6ab3123eec3e5226002aa0a0eb399b34ebdbff546d31ada478eea48b213496d` |
| 2026-07-22T06:11:27Z | +1 | 3715 | 1060 | 3928 | 7 | 45 | 26 | `866a4c1373725021dc909a931be137ba3acf55e9a610a7a3ba1db25306d77084` | `3669a630fbf400c5d0ef5848b7064ed07f2d15862eccd3071049733b8d3aad5a` |
| 2026-07-23T06:16:25Z | +34 | 3723 | 1077 | 3928 | 8 | 48 | 27 | `6561dfcbd57ae17133ca590394f3ff353ca265ca9b59f33e39bf4784cd2a3b7e` | `3b8f5bf9b26c9830f654bf03a89f6a9a666d8d4e04bb5f428315b85ff574c55a` |
| 2026-07-24T06:05:41Z | +3 | 3724 | 1077 | 3928 | 8 | 50 | 28 | `5fd0a0349e7bab83ee87067b63d945ddf6fbe557110ce8e27b6c12496772c51e` | `1ebaf9cd26e196f407f757ead069db0ad65602829fc7821a4eaef2540cf5c8fe` |
| 2026-07-25T06:05:55Z | +5 | 3724 | 1077 | 3928 | 8 | 52 | 29 | `b2c806a4d0788e251498f7f621f454993ebd8b8a42c490bda0fa9ebc05755547` | `ae789e891764514b80ebaec54e3e0adb8ead280e8c02d6a8a57dc3353ec025cc` |
| 2026-07-26T06:07:03Z | +7 | 3725 | 1078 | 3928 | 9 | 52 | 30 | `097b3d51cfc8bbd2baf7392d72661117334f4a91cd76f41203883000dc00a51a` | `ae789e891764514b80ebaec54e3e0adb8ead280e8c02d6a8a57dc3353ec025cc` |
| 2026-07-27T06:16:12Z | +17 | 3727 | 1083 | 3933 | 9 | 54 | 31 | `3e4366735959ac753d2a7083959ccd49e1971e80ae8921cd09278b10f5a322ad` | `2eafaaa77d3a7d3c8de5d4c413e04dc86e4d8df2865f32651134b5c322bd9c99` |
| 2026-07-28T06:03:54Z | +3 | 3727 | 1083 | 3933 | 9 | 54 | 31 | `2733ddcae206a053f3cde52cb554ff7f322040de9ededf609504f0837fa877fb` | `2eafaaa77d3a7d3c8de5d4c413e04dc86e4d8df2865f32651134b5c322bd9c99` |
| 2026-07-29T06:05:48Z | +2 | 3727 | 1083 | 3933 | 9 | 55 | 32 | `9a1dbe87a9c760446c307d871ba649258e513df3fb377c38468166951db8e77b` | `2eafaaa77d3a7d3c8de5d4c413e04dc86e4d8df2865f32651134b5c322bd9c99` |
| 2026-07-30T06:05:06Z | +6 | 3728 | 1084 | 3933 | 10 | 56 | 33 | `39d5aee316f129d7cc0d52cc792dae414456a5c110c41d6a43cb4ca159f20425` | `a1c2134f29c7056b1d142aff2b041e51b7bd26d1e0262e271b9911adbaf8be10` |
| 2026-07-31T06:05:09Z | +3 | 3728 | 1084 | 3933 | 10 | 58 | 34 | `4a6b8cd9d9bf22a2b36d891d9b7411e3b4f6d0c20a336869a378cfdce9c45a08` | `7c8bde0e6ea2e337c62a3d9001c4529b059d2557065d9001d933460c4fd4c6e2` |
| 2026-08-01T06:10:44Z | +47 | 3745 | 1113 | 3933 | 10 | 60 | 35 | `77f11e1fca517fa12ba47206e855e6cb717294d8a5476ca6f307383883ac41d0` | `5d9e5e6fd95db1d18b7bad3d0c9a959f88ce4d79c02f0e6644f63c48051fde8d` |
| 2026-08-02T06:05:58Z | +2 | 3746 | 1113 | 3933 | 10 | 61 | 36 | `bf64fe16eb8892acacf9640ac007c776cb1b85567314b38eb1cf842998843587` | `66cc382191782a2d08e0aa1620f1aa0e9797bf600561d25bc7e474becaff8dee` |
| 2026-08-03T06:29:01Z | +17 | 3749 | 1121 | 3938 | 10 | 63 | 37 | `633f27e55c419d9e62f1537037c093f7a38078c98643da7bdd0887ab4307657c` | `2f792f9fa22c062132508fa5b29e403bd598cb0ac794d44ed268ae66dcf549c0` |
| 2026-08-04T06:08:28Z | +3 | 3749 | 1121 | 3938 | 10 | 66 | 38 | `e0ca6fb6c8529ca141ee44f996e29b08070cde597774a0eea440e374dec78947` | `320205ff0ac430426e49d79efdfb0f6e45cba31b6cfdc4d78d0bdc879182c2d1` |
| 2026-08-05T06:20:28Z | +30 | 3754 | 1131 | 3949 | 10 | 68 | 39 | `5a3eb177be0197eed2020daf33bafa1d31cd4e5cba1617964936555ddc32ef46` | `329d195317d9a6df0b4a00e4af95390b2f0dbb21dabda270080ed74a5d9c9641` |
| 2026-08-06T06:05:51Z | +4 | 3754 | 1131 | 3949 | 10 | 69 | 40 | `5b9fa86b177a3da52afaface46f85c46191ce657c766564aa02be56cd75da125` | `d261856d9843e6cbe08ef541623c5126002e0b4ceb1904936d1e922b3b6b3207` |
