# FreeREAC — publishing plan and org cleanup

**Date:** 2026-09-01
**Status:** PROPOSAL — audit only. **Nothing here has been executed.**
**Authority:** every action below waits for the operator's explicit, per-repo sign-off.
No repo was archived, deleted, made public, renamed, pushed to or edited to produce
this document.

The org exists to **publish our findings and ship the tools**, and the key
deliverable is neither a capture nor a decompile — it is **the protocol,
explained**. So this is a publishing plan first and a cleanup list second: what
knowledge we hold, which public repo it belongs in, what has to happen before it
may go, and in what order. The lifecycle verdicts are kept because they answer a
question the publishing plan asks — *is this repo still a destination?*

## What was done to produce this

Read-only evidence only: `gh repo list FreeREAC`, `gh api` GETs against
`/orgs/FreeREAC`, `/repos/FreeREAC/<name>`, its `commits`, `issues`, `pulls`,
`branches`, `contents` and `git/trees` endpoints, and `git log` / `git rev-list` /
`git grep` against the local checkouts. No `PATCH`, no `POST`, no `DELETE`, no
`git fetch`, no `git push`, no settings write.

Local remote-tracking refs were cross-checked against the live default-branch SHAs
before any "unpushed" count below was trusted — all twelve tracked repos were in
sync, so the ahead-counts are real work and not a stale `origin/*`. Every sweep
that reports an **absence** was run with a positive control in the same command;
where that matters the control is named inline.

---

# Part 1 — The findings inventory

What we know, where it currently lives, and where it should be published.

| finding | lives now | target | sanitization needed | state |
|---|---|---|---|---|
| Head-amp op-`0403` = **Roland DT1 SysEx** + the staging/commit model | **already public** — `reac-protocol/wire-format.md` | — | — | **DONE** |
| **Wireshark dissector** (`wireshark/reac.lua`) | **already public** — `reac-tools` | — | — | **DONE** |
| **`reac.ksy`** machine spec + generated facts + CI conformance | **already public** — `reac-protocol/spec/`, with `ksy-conformance.yml` | — | — | **DONE**, actively maintained |
| **OHRCA / `cfea[19]` rate gate** | local branch `facts/cfea19-is-firmware-dependent-2026-08-29`, **6 commits unpushed** | `reac-protocol` | name the provenance — firmware behaviour cross-checked against rig capture | **STAGED.** Public has 1 mention in `wire-format.md`, 8 in `reac.ksy`; the sharpened *"the box does NOT simply follow the byte — it is firmware-dependent"* is **not public** |
| **REAC Mode switch M / S / SP** characterisation | local `reac-protocol` `main`, **2 commits unpushed** (`0028c75`, `c68dfd5`) | `reac-protocol` | name the source — `c68dfd5` reads the firmware's own FSM; cite it and the method | **STAGED. Public coverage is ZERO** — no published reac-protocol doc mentions the mode switch at all |
| **Enrolment law** (a channel stays silent until a head-amp record lands) | `reac-pw/docs/` (private) + `reac-lab` runbooks on an unpushed branch | `reac-protocol` | none — `reac-pw/docs/` is **free of Ghidra/decompile/ROM material** (verified, positive control passed), so this is wire-derived | **GAP.** Thin public trace (2 mentions in `wire-format.md`, 6 in `firmware-findings.md`); the law itself has never been stated publicly |
| **Head-amp decode module** `reac/headamp.py` (351 ln) + `capture-headamp.py` + `tests/test_headamp.py` | **local `reac-tools` lineage only** | `reac-tools` (published) | **none** — SPDX-headered, 0 forbidden tokens, 0 MAC literals | **READY — blocked by lineage** (see the law, §2) |
| **Control-plane decode** `reac/ctrl.py` (152 ln) + `tests/test_ctrl.py` | local `reac-tools` only | `reac-tools` (published) | **none** — clean by the same sweep | **READY — blocked by lineage** |
| **The observer — `reac.observe`** `reac/observe.py` (339 ln) + `tests/test_observe.py` (110 ln) | local `reac-tools` only | `reac-tools` (published) | **none** — clean by the same sweep | **READY — blocked by lineage** |
| Capture index `data/capture-index.{json,md}` + `tools/build_capture_index.py` | local `reac-tools` only | `reac-tools` **or** `reac-captures` (private) | an index of private captures may name rig material | **DECIDE** |
| Defect / convergence census | local `reac-protocol` branch `census/defects-2026-08-23`, 4 commits (subset of the cfea19 branch) | `reac-protocol` | — | **STAGED, unpushed** |
| `96K-WIFI-FINDINGS-2026-06-04.md` (8 KB) | loose file in the working dir | `reac-docs` | **none** — 0 forbidden tokens, 0 firmware refs | **READY** — wants one read-through + an SPDX/attribution header |
| **`REAC-PROTOCOL-AND-TESTS.md`** (459 ln) — the *wire* ground truth + rig test methodology | loose file, banner reads **"PRIVATE — keep OUT of git"** | **split**: wire half → `reac-protocol`; rig/test methodology → `reac-docs` | **2 lines carry the rig /24** — a sanitize-guard token, and the only real blocker. Its 16 firmware-RE citations are now provenance to keep, not material to remove | **NEARLY READY** — needs the split + that one scrub |
| **`REAC-PROTOCOL-FROM-SOURCE.md`** (623 ln) — the *source* half, from the SH-2 decompile | loose file, banner reads **"PRIVATE — keep OUT of git"** | **`reac-protocol/firmware-findings.md`** — this *is* the protocol explanation, and it publishes with its provenance named | drop the two **binary paths** it cites (`S-1608.BIN`, `M-300.PRG`) as fetch instructions and the `/tmp` decompile paths; keep the Ghidra/method/address citations as **provenance**. No binary is redistributed either way | **READY FOR A PASS** — the writing is done; it needs an editorial provenance header, not a rewrite |

**Two corrections to the staged-docs memory**, both verified against the live API:

- It records `reac-protocol` **PR #4** (DT1 SysEx + commit model) as *open, awaiting
  the user's merge*. PR #4 is now **`closed`, `merged=false`** — **but its content is
  live on `main` anyway**: the published `wire-format.md` carries the full DT1 SysEx
  section, marked `[V]`, at lines 364–382. The content landed by another route; the
  PR was closed rather than merged. Nothing is missing, but the memory's state line
  is stale.
- It flags `reac-tools` / `reac-label` / `reac-aes67` as sitting under `linuxnow`
  while `reac-docs` lists them as FreeREAC. **All three now resolve to FreeREAC**
  (`reac-label`'s local `origin` still *reads* `linuxnow/reac-label`, but GitHub
  redirects it to the transferred FreeREAC repo — the SHAs match).

---

# Part 2 — What publishes, and the one hard line

The deliverable is **the protocol, explained**. Everything else in this section
serves that: firmware RE and packet captures are *sources*, and a source is
something you **cite**, not something you hide.

## The rule

**Findings publish. Provenance is named. Vendor binaries are never redistributed.**

- **Firmware-RE findings publish, with their source and method cited.** If a
  conclusion came out of a decompile, say so: which device, which image version,
  which method (Ghidra headless, literal-pool read, disassembly), and what
  cross-check confirmed it on the wire. Naming the method is what makes the finding
  *checkable* by a reader — it is a feature of the document, not a leak from it.
  Addresses, function names and ROM constants are legitimate citation when they
  carry the argument.
- **Raw captures may be included where they support a finding.** A pcap that
  demonstrates the claim belongs beside the claim. Public fixtures already work this
  way — `reac-tools` ships `tests/fixtures/real_reac_stream.pcap`.
- **The one hard line: we never redistribute Roland's own firmware binaries.**
  `.BIN`, `.PRG`, RSFF containers, extracted ROM images — citing them, explaining
  them and quoting the behaviour they encode is fine; republishing the copyrighted
  binary is not. This is the single item in this plan with no discretion attached.

## Review flags, not laws

Two practical gates apply to captures. They are **operator review flags** — reasons
to look before pushing, never reasons a capture cannot publish.

- **Size.** The org is on the free git-LFS quota (~550 MB of 1 GB already used).
  A large corpus is a hosting question, not a permission question: prefer the
  *minimal slice that demonstrates the finding* over the whole session, and check
  the quota before adding bulk.
- **Performance audio.** Some captures carry real programme material from a live
  session. That is a thing the operator would reasonably want to review before it
  goes out — not a rights problem we have assessed, just a judgement call that is
  the operator's to make, per capture.

Neither flag applies to the synthetic and test-tone captures, which are the bulk of
what the findings actually rest on.

## What stays private, and why

Only one thing stays private for a reason of principle: **the vendor firmware
images themselves**. `reac-firmware-re` is where they live, and that is what keeps
it private — not its notes, not its analysis, not its decompile *conclusions*, all
of which are now publishable material with citation. The repo is the **binary
archive**; the findings it produced belong in `reac-protocol`.

`reac-captures` stays private as a **default, not a rule**: it is the working
corpus, it is large, and some of it carries performance audio. Individual captures
move out of it into public repos whenever they support a published finding and the
operator has cleared them under the two flags above.

## The one genuine can't-push, unchanged

**The `reac-tools` LOCAL lineage never pushes.** This one has nothing to do with
Roland and does not relax. Its own README says so under the heading *"This lineage
must never be pushed to FreeREAC"*: the local tree descends from the pre-migration
root, the published repo was **re-rooted from scratch**, the two share **no common
ancestor**, and the old history still carries **a live root password at a reachable
blob**. A sanitising commit on top does not remove a blob. Material that belongs
upstream moves by **copying the content onto a branch off the published `main`** —
never by push, merge or rebase of this lineage. The local tree also holds the
numpy/scipy scripts that belong in `reac-analysis`: present here on purpose, absent
from the published repo on purpose.

**And one routing trap:** `reac-aes67`'s `origin` is the **private history archive**
(`linuxnow/reac-aes67`, the leaky pre-sanitisation history). The FreeREAC remote is
the separate `fr` remote. A bare `git push origin` sends work to the archive, not to
the org — always name `fr`.

## Still true regardless: the site-token scrub

The sanitize-guard token set (venue and client names, the rig /24) is unrelated to
any of the above and still applies to every public push. It is about **our** third
parties, not Roland's. It is the reason `REAC-PROTOCOL-AND-TESTS.md` cannot move
until two lines are fixed — the only hard blocker left on that document.

## A thread closed, so nobody reopens it

**`reac-analysis` does NOT carry the `reac-tools` credential.** It publishes
`reac-measure.sh`, and its history contains a commit named *"drop the embedded
credential"* sitting on top of an earlier import — exactly the shape that means a
leak. It is not one: at the pre-scrub commit `6680969` the variable already reads
`PW="REDACTED"` and the host defaults to a `ROUTER_IP` placeholder. That lineage was
sanitised **before** publication. Verified with a working positive control — the
same grep does find the `sshpass` line at that ref — so the negative is real.

---

# Part 3 — The tools roster

What the org offers, or should, and the build state of each.

| tool | repo | what it is | build state |
|---|---|---|---|
| **libreac** | `libreac` (public) | shared C protocol library — mode descriptors, rate detection, frame + geometry helpers | **SHIPPING.** 0.7.1 header-only geometry helpers on `main` (08-30), ABI unchanged. 3 unpushed local commits on `feat/rpm-0.6.0`. **No `libreac` RPM in the package repo despite its README claiming one.** |
| **reac-tools** (published lineage) | `reac-tools` (public) | stdlib-only traffic analysis + the Wireshark dissector | **SHIPPING but INCOMPLETE.** Three finished, clean modules — `observe`, `ctrl`, `headamp` — exist only in the un-pushable lineage. |
| **reac-analysis** | `reac-analysis` (public) | numpy/scipy signal bench — pitch, clock wobble, spectrum, glitch, PLC | **SHIPPING, dormant by design.** Its last commit is the 07-29 split itself, not decay. **No local checkout exists** — clone before touching it. |
| **reac-aes67** | `reac-aes67` (public) | REAC → AES67 (RTP L24) converter, OpenWrt apk + LuCI | **SHIPPING** (apk v0.2.1). Product-line question open — see the DECIDE verdicts. 4 unpushed commits ahead of `fr/main`. |
| **reac-repacer** | `reac-repacer` (public) | transparent L2 de-jitter / re-pacing relay, apk + LuCI | **SHIPPING** (apk v0.2.3). 11 unpushed local commits (6 on `main`, 5 on the revalidation branch). |
| **reac-transport** | `reac-transport` (public) | VLAN trunk / gretap transport, apk | **SHIPPING** (apk v0.1.0); its VID convention is partially superseded. 3 unpushed. |
| **reac-label** | `reac-label` (public) | V-Mixer / M-5000 → channel-name labeller | **SHIPPING.** Its only consumer is `reac-aes67`; its fate follows that decision. 4 unpushed. |
| **the observer — `reac.observe`** | target `reac-tools` | 339-line module + a 110-line test suite | **FINISHED AND CLEAN, BUT UNPUBLISHABLE BY ITS CURRENT PATH.** Lineage-blocked; the transport is the work, not the code. |
| **reac-pw** | `reac-pw` (**private**) | PipeWire-native REAC endpoint — the fabric as graph nodes | **SHIPPING PRIVATELY.** 435 commits, 14 open issues. Gated on the console milestone, not on hygiene — see Part 5. |
| **OBS source plugin** | — | a native REAC source for OBS | **ASSESSMENT IN FLIGHT.** A parallel lane is evaluating the route; this plan deliberately does not duplicate that work. Two data points already on record: `reac-docs` local `main` carries an unpushed commit *"OBS reaches PipeWire audio when the separate plugin is installed, and this rig has it"*, and a local `obs-h8819-source` checkout exists with a modified `CMakeLists.txt`. **Leave this row to the parallel lane.** |

**The gap the roster exposes:** the org's front page describes a four-box OpenWrt
stack and names none of the three things that are actually shipping hardest —
`libreac`, `reac-pw`, and the package repository. See Part 7.

---

# Part 4 — The publishing sequence

Each line is one operator gesture. Order matters: waves are gated, items inside a
wave are independent.

### Wave 0 — zero sanitization risk; push as-is

These commits were authored in their own clean repo, are GPG-signed, and pass the
repo's own sanitize-guard. Nothing in them has ever touched the private material.

1. **`reac-protocol` `main`** — 2 commits, both go. `0028c75` (mode switch = the
   splitter clock role) is wire observation; `c68dfd5` reads the firmware's own FSM
   and publishes **with that source named**. This is the mode-switch characterisation,
   of which the public repo currently says nothing at all.
2. **`reac-repacer`** — 11 commits, docs and tests only.
3. **`reac-lab` / `reac-label` / `reac-transport`** — 13 commits between them on
   `revalidation/2026-08-23`, all doc hygiene.
4. **`reac-docs` `main`** — 5 commits; the OBS commit may want the parallel lane's
   verdict first.
5. **`reac-firmware-re` `main` → the PRIVATE remote** — 64 commits. Highest orphan
   risk in the org; **zero** publication risk because the destination is private.
6. **`reac-captures` `main` → the PRIVATE remote** — 11 commits, plus six untracked
   capture files to decide on.

### Wave 1 — needs a review pass, no new work

7. **`reac-protocol` `facts/cfea19-...`** — 6 commits, the OHRCA rate gate. One
   editorial read to name the provenance, then push + PR. This is the
   highest-value unpublished finding we hold: the byte gates the drivable rate
   class, and the box does not simply follow it.
8. **`96K-WIFI-FINDINGS-2026-06-04.md` → `reac-docs`** — token-clean; needs a
   read-through and an SPDX/attribution header.
9. **`libreac` `feat/rpm-0.6.0`** — 3 commits; decide at the same time whether the
   RPM ships to the package repo, whose README already promises it.

### Wave 2 — needs work before it can move

10. **`reac.observe` + `reac/ctrl.py` + `reac/headamp.py` + their tests → published
    `reac-tools`**, by **copying the content onto a branch off the published `main`**
    (Part 2). The files are already clean and SPDX-headered; the *transport* is the
    work. This is the single biggest ready-but-unshipped block of tooling we have.
11. **The enrolment law → a `reac-protocol` section.** Source is `reac-pw/docs/`,
    verified free of firmware-RE material, so it is publishable wire knowledge. It
    has never been stated publicly.
12. **`REAC-PROTOCOL-AND-TESTS.md` → split**, wire half to `reac-protocol`, rig/test
    methodology to `reac-docs`. **Two lines carry the rig /24** — scrub before
    either half moves.
13. **`REAC-PROTOCOL-FROM-SOURCE.md` → `reac-protocol/firmware-findings.md`.**
    This is the protocol explanation, and under the Part 2 rule it publishes with
    its provenance named — device, image version, method, and the wire cross-check.
    The editorial pass is small: add a provenance header, and turn the two vendor
    **binary paths** and the `/tmp` decompile paths into citations rather than fetch
    instructions. **No firmware binary moves.** Promote this out of Wave 2 if the
    operator wants the explanation out early — it is closer to ready than its
    position here suggests.
14. **`reac-pw` publication** — gated on the console milestone (Part 5), then its
    six readiness items.
15. **Front-page and package-repo corrections** (Part 7) — small, and they change
    what a visitor can find.
16. **The OBS plugin** — blocked on the parallel assessment, not on us.

---

# Part 5 — `reac-pw` publication readiness

`reac-pw` is private while `libreac` is public, and the org's own package repo
advertises `reac-pw` RPMs. So the question is live. What the publishing rules
require, and where the repo stands — measured, not assumed.

**The governing constraint is not hygiene.** Commit `4a6001c` (2026-06-15) records
the operator decision verbatim: *keep the JOIN/HOLD handshake (`reac_ctrl` /
`reac_fsm`) private until it is validated against a real M-5000 console — revisit
publishing then.* Until that milestone is called, everything below is preparation,
not a green light. **Nothing here proposes flipping visibility.**

**Identity — PASS.** All 453 commits across all refs are authored
`Pau Aliagas <linuxnow@gmail.com>`; zero from any other identity. The repo already
pins `user.email`, `user.signingkey A14B3E1E1F69EBF4` and `commit.gpgsign=true`.

**GPG-signed history — 429 of 435 signed on `main` (6 gaps).** Six consecutive
commits dated 2026-08-23 verify `N` (`4bba369`, `f688900`, `d5a5cdc`, `d1118fe`,
`cd20efd`, `cfc27cc` — the SENS-sweep rig-data run). Every other commit verifies
`G`. Re-signing needs `filter-branch --commit-filter`, which rewrites SHAs; whether
six gaps in 435 commits is a blocker is an operator call.

**License — PASS with a gap.** `LICENSE` (full GPLv3) and `NOTICE` are present and
GitHub detects GPL-3.0. Per-file SPDX headers cover **133 of 147** tracked sources.
The 14 without one are all tooling, none of it core:
`tools/{build.sh,gen-scene-body.py,probe-ports.sh,recover-scene.py,rig-restart-master.sh,scene-on-wire.py,sine-level.py,tone-band.py,tone-purity.py,wav-rms.py,wire-audio.py}`,
`tests/{packaged-shape-starts.sh,test_sink_buffer_pairing.py}`,
`docs/rig-data/2026-08-23-clock/runab.sh`.

**Sanitization tokens — PASS, tree and history.** The forbidden set returns **zero**
hits in the working tree and **zero** across all refs by pickaxe. Both sweeps ran
with a positive control in the same command — the tree grep finds `REAC` in 192
files, the history pickaxe finds `reac_ctrl` in 88 commits and `0x8819` in 33 — so
the empty results are absence, not a broken search.

**Firmware-derived material — none present.** `reac-pw/docs/` contains no Ghidra
reference, decompile listing, `FUN_########` symbol or ROM load address. Positive
control: the same pattern does match both loose `REAC-PROTOCOL-*` docs, so the
pattern works. Under the Part 2 rule this is no longer a gate — it just means the
enrolment law can be lifted from these docs as pure wire observation, with no
provenance note needed.

**Secrets — PASS.** The only `password` match in tree or history is the GPL text
inside `LICENSE`. No private-key material at any ref.

**Blockers that are real, and cheap:**

1. **No CI sanitize guard at all.** `reac-pw` has **no `.github/` directory**. Every
   other public FreeREAC repo carries `sanitize-guard.yml`. Publishing without it
   removes the one mechanical check keeping the token set at zero — and it is at
   zero today only because nothing has re-introduced it.
2. **A scratchpad path with an AI trace and a personal home path.**
   `docs/rig-data/2026-08-23-clock/runab.sh:5` hardcodes a `/tmp/claude-1000/-home-pau-...`
   directory — two rules broken in one line.
3. **`docs/RIG-MASTERS.txt:9` names `CLAUDE.md`** in prose. A direct AI trace.
4. **Two files use "adversarial"** in the banned sense —
   `docs/OHRCA-UPSTREAM-DUPLICATE-FRAMES.md:110` and `docs/STATUS-2026-06-14.md:39`.
5. **A compiled artefact is tracked:** `tools/__pycache__/recover-scene.cpython-314.pyc`.
6. Ten `reac-pw-*.tar.gz` tarballs and a `logs/` tree sit in the working directory;
   the tarballs are untracked, the two `logs/*.log` files are not.

Items 2–4 need a **history** pass, not just a tree edit, for the same reason the
`reac-tools` re-root was necessary. No status emoji and no `(C, TDD)` tags were
found anywhere; the one commit message matching "generated with" is a false
positive (*"a body generated with the M-200i's MAC"*).

---

# Part 6 — Repo lifecycle

The publishing plan needs to know which repos are still destinations. They all are —
but four of them look dead on GitHub for a reason that is not death.

## Delta since the 2026-08-29 listing

**No repo appeared and no repo vanished.** The org still holds exactly 16 repos —
12 public, 4 private, **0 archived** — the same 16 names recorded on 2026-08-29.
Three moved: `reac-pw` 08-29 → **08-31**, `reac-protocol` 08-23 → **08-30**,
`libreac` 08-29 → **08-30**.

One correction to the older org reference note: it records `reac-firmware-re` as
public. **It is private**, as the private-RE-repos note says and the live API confirms.

## The table

`push` = repo `pushed_at` (any branch). `tip` = default-branch head date — where the
two differ, the newer push went to a **branch**, not to `main`. `local` = commits in
the local checkout and absent from the remote.

| repo | vis | push | main tip | iss | PR | local unpushed | verdict |
|---|---|---|---|---|---|---|---|
| `reac-pw` | private | 08-31 | 08-31 | 14 | 0 | 1 on main + 7 branches | KEEP-ACTIVE |
| `reac-protocol` | public | 08-30 | 08-30 | 0 | 0 | 2 on main + 2 branches (10) | KEEP-ACTIVE |
| `libreac` | public | 08-30 | 08-30 | 0 | 0 | 3 on `feat/rpm-0.6.0` | KEEP-ACTIVE |
| `reac-captures` | private | 08-23 | 08-23 | 0 | 0 | **11 on main** (to 08-31) | KEEP-ACTIVE |
| `reac-docs` | public | 08-23 | **06-14** | 0 | 1 | **5 on main** + 2 branches | DECIDE |
| `reac-repacer` | public | 08-21 | 08-21 | 0 | 0 | **6 on main** + 5 on a branch | KEEP-ACTIVE |
| `reac-aes67` | public | 07-29 | 07-29 | 0 | 0 | **4** (ahead of `fr/main`) | DECIDE |
| `reac-tools` | public | 07-29 | 07-29 | 0 | 0 | 34 ahead / 17 behind — **unrelated lineage, never push** | KEEP-ACTIVE |
| `reac-analysis` | public | 07-29 | 07-29 | 0 | 0 | no local checkout | KEEP-ACTIVE |
| `freereac.github.io` | public | 07-28 | 07-28 | 0 | 0 | no local checkout | KEEP-ACTIVE |
| `reac-firmware-re` | private | 07-21 | **06-15** | 0 | 2 | **64 on main + 7 branches** | KEEP-ACTIVE |
| `audio-rig-mcp` | private | 07-21 | **07-04** | 0 | 0 | 0 | DECIDE |
| `reac-transport` | public | 06-14 | 06-14 | 0 | 0 | **3** (on a branch) | DECIDE |
| `reac-lab` | public | 06-14 | 06-14 | 0 | 0 | **6** (on a branch) | DECIDE |
| `.github` | public | 06-14 | 06-14 | 0 | 0 | 0 | KEEP-ACTIVE (content stale) |
| `reac-label` | public | 06-14 | 06-14 | 0 | 0 | **4** (on a branch) | DECIDE |

**Counts: KEEP-ACTIVE 10 · DECIDE 6 · ARCHIVE-CANDIDATE 0.**

## No repo is archivable today

The 2026-08-29 note nominated four repos as looking superseded. **All four carry
unpushed local commits dated 2026-08-23** on a `revalidation/2026-08-23` branch that
was never pushed — and in every case the commits are exactly the doc-hygiene work an
archive is supposed to preserve:

- **`reac-lab`** — 6 commits, incl. *"docs: say which repo wins, before the reader
  opens a dated file"* and *"docs: banner the dated specs whose central claim is
  refuted"*. Archiving first freezes the repo in the state those commits exist to fix.
- **`reac-label`** — 4 commits (README framing, NOTICE, docstrings).
- **`reac-transport`** — 3 commits (README segment sizing, MTU arithmetic).
- **`reac-repacer`** — 5 on the same branch **plus 6 more on local `main`**.

A repo that looks stale on GitHub because its August work never left the laptop is
not stale; it is **unpushed**. So there is **no ARCHIVE-CANDIDATE here**. The archive
question reopens per repo only after its local work is landed or explicitly
abandoned — that ordering is the whole point, and it is why Wave 0 exists.

Two repos are far out of sync the same way and are the largest orphan risk in the
org: **`reac-firmware-re`, 64 unpushed commits on local `main`** (remote last moved
2026-06-15, local runs to 08-25) plus seven local branches of which only two have
open PRs; and **`reac-captures`, 11 unpushed commits** to 2026-08-31.

## Verdicts, with the evidence

**KEEP-ACTIVE.** `reac-pw` (the product; 260 files in the openmixer tree name it) ·
`reac-protocol` (the reference; 27 files) · `libreac` (three inbound consumers; 53
files) · `reac-captures` (the ground truth, growing) · `reac-repacer` (openmixer's
design notes: *"this is where rate matching belongs"*) · `reac-tools` (**flag, do not
fix** — Part 2) · `reac-analysis` (dormant by design, not decayed) ·
`freereac.github.io` (the `dnf` repo the openmixer install path uses — not
archivable) · `reac-firmware-re` (private because it holds the vendor **binaries**;
its findings publish, and 64 commits need it as a push target) · `.github` (keep; content stale).

**DECIDE** — each with the question that decides it:

- **`reac-docs`** — *land the branches, or accept that findings now live in
  `reac-protocol` and `reac-lab`?* Remote `main` last moved 06-14; the 08-23 push was
  a branch. PR #1 has been open since 2026-06-30. Local `main` already merged both
  branches and is 5 ahead. The current state says neither thing.
- **`reac-aes67`** — *is the AES67 line still a product, or has `reac-pw` replaced
  it?* openmixer's ADR is literally `0001-native-reac-not-aes67.md`, and the local
  unpushed commits say *"reac-pw has its own repo, so this tree stops presenting
  itself as its source"*. But it is the OpenWrt-router product with its own apk
  releases. A product decision, not a hygiene one.
- **`reac-label`** — *it feeds `reac-aes67`; its fate follows.*
- **`reac-transport`** — *superseded by the trunk/VLAN spec?* openmixer records its
  VID convention as **"demoted to available prior art, not a requirement"**. That is
  supersession of the *convention*, not of the OpenWrt package.
- **`reac-lab`** — *has everything found a home yet?* openmixer still cites it as the
  provenance store — *"holds what has no home yet; check what has since found one"*.
  That check is the archive precondition.
- **`audio-rig-mcp`** — *superseded by the harvest-MCP knowledge repo?* The only repo
  with **zero** inbound references from the openmixer tree; no local work, no
  branches, 17 KB, `main` tip 07-04. The strongest archive case in the org — and,
  being private and tiny, the lowest-value one to act on.

---

# Part 7 — Front-page and packaging defects

Small, and they decide what a visitor can find.

- **The org profile README is stale.** `.github/profile/README.md` lists eight repos
  and omits three public ones: **`libreac`** (the shared library three tools depend
  on), **`reac-analysis`**, and **`freereac.github.io`** (the package repository).
  Its ASCII stack diagram mentions neither `libreac` nor `reac-pw`. **A visitor
  cannot discover the package repo from the org front page.**
- **The package repo over-declares.** `freereac.github.io`'s README says it serves
  *"signed RPMs for the openmixer console and the REAC transport stack (`openmixer`,
  `reac-pw`, `libreac`)"*. The published tree holds **only** `openmixer*` RPMs —
  `openmixer`, `-server`, `-full`, `-web-ui`, four plugin sets and an SRPM. There is
  **no `reac-pw` RPM and no `libreac` RPM** under `rpm/fedora/44/`. Either ship them
  or narrow the sentence; a declaration a client cannot detect as false is the
  failure mode to avoid.
- **`reac-pw` has 45 stale branches on the remote.** Recorded as an observation only.
  **No bulk branch deletion is proposed.** If any are removed it is one at a time, by
  name, after the operator has looked at each.

---

# Part 8 — The commands, for the operator to run

**None of these has been executed.** Each is listed against the sign-off it needs.
`gh` must be authenticated as the org owner.

### Wave 0 — read the log, then push (one repo at a time)

```
git -C ~/Devel/audio/reac-protocol    log --oneline origin/main..main
git -C ~/Devel/audio/reac-repacer     log --oneline origin/main..main
git -C ~/Devel/audio/reac-docs        log --oneline origin/main..main
git -C ~/Devel/audio/reac-lab         log --oneline origin/main..revalidation/2026-08-23
git -C ~/Devel/audio/reac-label       log --oneline origin/main..revalidation/2026-08-23
git -C ~/Devel/audio/reac-transport   log --oneline origin/main..revalidation/2026-08-23
git -C ~/Devel/audio/reac-repacer     log --oneline origin/main..revalidation/2026-08-23
```

Then, per approved repo:

```
git -C ~/Devel/audio/<repo> push origin main
git -C ~/Devel/audio/<repo> push origin revalidation/2026-08-23
```

**Never** for `reac-tools` (Part 2 — the un-pushable lineage). For `reac-aes67`
the remote is `fr`, not `origin` — a bare `push origin` reaches the private archive:

```
git -C ~/Devel/audio/reac-aes67 log --oneline fr/main..HEAD
git -C ~/Devel/audio/reac-aes67 push fr HEAD:refs/heads/<branch>
```

The two largest orphan risks, both to **private** remotes:

```
git -C ~/Devel/audio/reac-firmware-re push origin main
git -C ~/Devel/audio/reac-captures    push origin main
```

### Wave 1 — the OHRCA rate gate, after the provenance read

```
git -C ~/Devel/audio/reac-protocol log -p origin/main..facts/cfea19-is-firmware-dependent-2026-08-29
git -C ~/Devel/audio/reac-protocol push origin facts/cfea19-is-firmware-dependent-2026-08-29
gh pr create --repo FreeREAC/reac-protocol --base main --head facts/cfea19-is-firmware-dependent-2026-08-29
```

### Wave 2 — moving the observer / ctrl / headamp by CONTENT (Part 2)

Clone the **published** lineage fresh — never reuse the local `reac-tools` checkout:

```
git clone https://github.com/FreeREAC/reac-tools.git ~/Devel/audio/reac-tools-pub
git -C ~/Devel/audio/reac-tools-pub checkout -b feat/observe-ctrl-headamp
cp ~/Devel/audio/reac-tools/reac/{observe.py,ctrl.py,headamp.py} ~/Devel/audio/reac-tools-pub/reac/
cp ~/Devel/audio/reac-tools/tests/{test_observe.py,test_ctrl.py,test_headamp.py} ~/Devel/audio/reac-tools-pub/tests/
cp ~/Devel/audio/reac-tools/capture-headamp.py ~/Devel/audio/reac-tools-pub/
```

Then commit in the fresh clone and push that branch. `cp` — not `git cherry-pick`
across the two repos: the point is that **no object from the old lineage is carried**.

### Publishing a capture alongside a finding (Part 2 review flags)

Check the size and glance at what the capture carries before it goes:

```
ls -lh ~/Devel/audio/reac-captures/captures/<name>.pcap
capinfos ~/Devel/audio/reac-captures/captures/<name>.pcap | head -20
```

Then copy the chosen slice into the public repo beside the doc that cites it —
never bulk-copy the corpus.

### Open PRs to triage

```
gh pr view 1 --repo FreeREAC/reac-docs
gh pr view 1 --repo FreeREAC/reac-firmware-re
gh pr view 3 --repo FreeREAC/reac-firmware-re
```

### If `audio-rig-mcp` is signed off for archiving

```
gh repo archive FreeREAC/audio-rig-mcp --yes
```

### If `reac-lab` / `reac-label` / `reac-transport` are signed off — AFTER Wave 0 lands their branches

```
gh repo archive FreeREAC/reac-lab --yes
gh repo archive FreeREAC/reac-label --yes
gh repo archive FreeREAC/reac-transport --yes
```

Archiving is reversible (`gh repo unarchive`) and leaves content readable, so
cross-repo documentation links keep resolving. It does **not** delete.

### `reac-pw` visibility — do NOT run without the console-milestone decision

Listed for completeness only. `4a6001c` gates this, and Part 5's items 1–4 are unmet.

```
gh repo edit FreeREAC/reac-pw --visibility public --accept-visibility-change-consequences
```

### Content fixes (ordinary PRs, no lifecycle change)

- `.github/profile/README.md` — add `libreac`, `reac-analysis`,
  `freereac.github.io`; put `libreac` and `reac-pw` in the stack diagram.
- `freereac.github.io` README — narrow the RPM claim, or ship the missing RPMs.
- `reac-pw` — add `sanitize-guard.yml`, SPDX-header the 14 files, untrack the
  `.pyc`, clear the four AI traces in tree **and** history.
