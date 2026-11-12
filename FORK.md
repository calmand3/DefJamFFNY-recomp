# Fork instructions: calmand3/DefJamFFNY-recomp

This repository is a fork of `vyanhursky/DefJamFFNY-recomp`, taken at `v1.2.0-beta.2` (b9b6201, 2026-11-12).
`AGENTS.md` and `PROGRESS.md` are the upstream project's files and describe its owner's (Vlad's) process.
**This file is the fork's own instruction set. Where it and `AGENTS.md` disagree, this file wins.** Everything
`AGENTS.md` says that is not changed here (tools, playbooks, the working loop, the hygiene rules) still applies.

Keep this file short. Edit it in the same turn as any change to goals, rules or state.

## 1. Purpose

Upstream's goal is a faithful, shippable port. This fork's goal is different: **use the recompiled game as a
platform and push it as far as AI-assisted engineering can go.** Because the game is C that we compile, any
function can be hooked, replaced or extended, and fixed limits can be lifted. The fork explores new content and
new systems, and accepts experiments that upstream would not ship.

## 2. Tracks

Work is organised into four tracks. The order below is the default priority (cheapest proven path first); the
owner can reorder it.

| Track | Starting point in this tree | First step |
|---|---|---|
| A. New fighters | Fighter Mod Support, milestones M1 to M5 (`src/patches/fighter_*.c`, `docs/fighter-mod-support.md`, `docs/fighter-mod-authoring.md`) | Build and load one original fighter pack end to end; record every limit hit |
| B. New animations and moves | Animation bank records and curve codecs are documented (`docs/research/fighter-animation-*.md`, `fighter-authored-animation-2026-11-05.md`) | Author one new animation and play it on a retail fighter |
| C. New venues | Nothing built. `docs/research/mod-system-design.md` rates it a long shot (fixed tables, memory budgets, save format) | Read-only spike: how `venue.xml` and venue resources load, and where the venue count is fixed |
| D. Online play | Nothing built. Related evidence: `docs/research/fight-determinism.md`, `crowd-nondeterminism.md`, `deck-determinism-98.md` | Spike: is a fight reproducible from seed plus inputs on one machine, then across two? |

Track D plan, in order, each step gated on the one before:
1. Determinism: same seed and same inputs give the same fight, measured with the existing replay tools.
2. Input-delay netplay for two players over a direct connection (both sides simulate, only inputs travel).
3. Rollback: save and restore of guest state within a frame budget. Attempt only after 1 and 2 hold.

Other ideas (new modes, new match types, AI-driven opponents) get a row here when the owner asks for them.

## 3. What changes from `AGENTS.md`

- **Owner.** Where `AGENTS.md` says "Vlad" as the person who approves, read "the fork owner" (GitHub `calmand3`).
- **Nothing goes upstream without the owner asking.** No pull request, issue or comment on any `vyanhursky/*`
  repository. Every pull request is opened with base repository `calmand3/DefJamFFNY-recomp`. GitHub's form
  defaults a fork's pull request to the upstream repository: check the base before submitting.
- **`PROGRESS.md` is read-only here.** It is upstream's record; editing it makes every upstream sync conflict.
  Read it with `python scripts/progress.py`. Fork state lives in §6 of this file and in `docs/fork/`.
- **Decisions** made in this fork are numbered `F<n>` in `docs/fork/decisions.md` (date, decision, reason), so
  they never collide with upstream's `D<n>`.
- **Research reports** for the fork go in `docs/fork/research/<topic>-<YYYY-MM-DD>.md`. One per spike, with the
  evidence, what was not verified, and the next step.
- **Releases.** The fork does not publish releases, and the upstream release process (`docs/releasing.md`, the
  nine-asset rule, release trains) does not run here unless the owner decides to publish.
- **Toolkit.** `tools/xboxrecomp` is pinned to upstream's fork of the toolkit, which we cannot push to. Prefer
  game-side changes in `src/`. If a toolkit change is unavoidable, stop and ask: it means forking the toolkit to
  `calmand3/xboxrecomp` and repointing `.gitmodules`.
- **Regression.** `python scripts/regress.py --quick` plus the focused checks for the area changed is the bar
  for a fork change. The full run is for a moment the owner picks, not for each change.

## 4. What does not change

- **Hygiene (`AGENTS.md` §3) applies in full.** No disc images, XBEs, game archives, lifted C, logs or built
  game executables in the repository, ever. No third-party copyrighted models, likenesses, music or artwork
  either: content made for this fork is original or supplied by the owner. Run `git add -A --dry-run` before
  every commit.
- Never name where a dump came from. It is "the owner's own dump".
- Game runs are serial. Kill the game before building.
- A sub-agent's report is evidence, not fact. A structural parse is not a native run.

## 5. Rules for experiments

- New behaviour lives in `src/patches/` (one file pair per feature), is **off by default**, and has a setting or
  environment variable, so the stock game stays testable and upstream merges stay small.
- Avoid editing upstream files when a new file will do. Small, separate changes merge cleanly; rewrites do not.
- State what was actually run. "Compiles", "unit tests pass" and "played a fight on Windows" are three different
  claims; say which one applies.
- Sync from upstream only when the owner asks: add the `upstream` remote, merge `upstream/main` into a branch,
  run the quick regression, and list conflicts in the pull request.

## 6. Current state and hand-off

Rewrite this section at the end of each session.

- 2026-11-12: fork created at upstream `v1.2.0-beta.2`. This file added. No fork code changes yet.
- 2026-11-12: owner is on Windows with their own disc image; build tools installing. First project is Track A:
  one original fighter, mesh generated with the owner's Meshy subscription. Static read of the model compiler:
  `docs/fork/research/model-compiler-read-2026-11-12.md` (no generic-mesh importer exists; a converter is needed).
- **Where work can run.** A cloud agent session has no dump and cannot build or run the game: it can read,
  research, design, and write code and unit tests. Building and running need the owner's computer with their
  own dump (`docs/build-and-play.md`, `docs/build-macos-linux.md`).
- **Next:** get the stock build running on the owner's computer; make an inheriting fighter pack to prove the
  install loop; export one retail donor and print its geometry and skin metadata; then write the GLB converter.

## 7. Open questions for the owner

1. Priority order of tracks A to D: is the default order in §2 right?
2. Answered 2026-11-12: Windows. The dump is not yet verified against `config/dump-manifest.json`.
3. Is this fork meant to stay public? That decides how strictly content sources must be vetted before commit.
4. Should fork work ever be offered back to upstream, or is this a separate line for good?
