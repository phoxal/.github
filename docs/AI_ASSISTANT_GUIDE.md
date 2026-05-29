# AI Assistant Guide

This is the canonical guide for AI coding agents (Claude Code, Codex,
Copilot, Cursor, …) working in any Phoxal repository.

Read this file first. Then read the repo-local `AGENTS.md` for that
repository's specific rules. The repo-local file wins where it disagrees.

## Project identity

Phoxal is a Rust framework for production-oriented autonomous mobile robots.
The product is a coherent stack — manifest-driven, simulation-first, typed
bus, container-based runtime topology — taking a robot from authored
`robot.yaml` to running, with the same conventions end to end.

See [VISION.md](VISION.md) for the principles, [ARCHITECTURE.md](ARCHITECTURE.md)
for the system model, [BLUEPRINT.md](BLUEPRINT.md) for the v1 autonomy
target, and [REPOSITORY_MAP.md](REPOSITORY_MAP.md) for the cross-repo layout.

## Architecture rules to internalize before editing code

These are not opinions to balance against other opinions — they are the
shape of the codebase.

- **Manifests are authoritative.** `robot.yaml`, `structure.urdf`, and
  `components/<name>/component.yaml` are the single source of truth. Runtimes
  parse them directly via the public schema crates. **Do not** synthesize a
  side-channel `runtime.config.json` / `phoxal-runtime.yaml` / generated
  configuration file.
- **Typed contracts only.** Every cross-process payload, query, or command
  belongs to a published `phoxal-*-api` crate. **Do not** publish ad-hoc JSON
  blobs over the bus. If it isn't typed, it isn't a contract.
- **Static topology.** The CLI's `PlatformRuntimeCatalog` is the topology
  source of truth. **Do not** add a runtime discovery / introspection
  protocol, a `--phoxal-describe` flag, a `RuntimeDescriptor` type, or a
  compatibility manifest. Bus decode errors are the mismatch-detection layer.
- **Coherent platform release.** All `phoxal-runtime-<name>-api` crates +
  all runtime images ship at the same workspace version. **Do not** mint
  per-runtime independent semver tracks. API evolution happens inside
  `pub mod vN` modules within a release.
- **Two distinct runtime layers.** Platform runtimes are the mandatory
  framework substrate (can be versioned/overridden, not removed). User
  runtimes are additive only. **Do not** add `enabled: false`, `skip`, or
  `exclude` semantics to platform runtimes.
- **Container ↔ native split.** Bus participants that ship as images run
  in docker; hardware drivers, Webots integration, and operator tools run
  native on the host. The bus is the only shared surface.
- **Engine ↔ CLI boundary.** Resolver, `phoxal.lock`, command
  implementations, native-process orchestration → CLI. `Runtime` trait,
  bootstrap, runtime context, logical clock, presence, bus integration →
  engine. Schema crates are shared. **Do not** reach across this boundary.
- **No backwards-compatibility shims for in-flight contracts.** This is
  pre-1.0. Breaking changes are loud, not quiet.
- **No ROS-compat layer.** Not `rosbridge`, not `tf2` parity, not
  `geometry_msgs` imitation.

## Repository ownership

Use [REPOSITORY_MAP.md](REPOSITORY_MAP.md). The short answer: a change belongs
in the repo that owns the contract the change crosses. Cross-runtime payload
change → `phoxal/framework`. Authoring schema change → `phoxal/framework`
schema crates. CLI behavior → `phoxal/phoxal-cli`. Component definition →
the `phoxal/component-<name>` repo.

When a change spans multiple repos, the API crate's repo lands first; the
consumers follow from crates.io / GHCR.

## How to reason about a change

For every code change, the agent should be able to answer:

1. **Which crate owns this concern?** If you can't name one cleanly, the
   change is probably structured wrong.
2. **Local behavior, or framework convention?** Local lives in the owning
   crate's `README.md`. Convention lives in the org-level docs (this repo).
3. **New or updated owner-local API contract?** If yes, the
   `phoxal-*-api` crate update goes first, then consumers.
4. **Does this need model/component validation?** If yes, the `validate`
   path covers it; add a test.
5. **Does it affect generated artifacts from `phoxal-cli`?** (`phoxal.lock`,
   `.phoxal/run/`, `.phoxal/cache/state.yaml`.) If yes, the CLI's resolver
   tests cover it.
6. **Deterministic under simulation logical time?** If the change touches
   anything time-sensitive, drive it from engine clock primitives, not wall
   time.

## Required test coverage

Carry these categories forward in every owning crate:

- **Contract drift.** Every API crate has at least one test asserting its
  `SCHEMA_NAME` and `SCHEMA_VERSION`.
- **Revision linkage.** Any consumer taking `map_revision` / `localize_revision`
  has both a mismatch-rejection test and a round-trip test.
- **Timestamp discipline.** A payload introducing `measured_at_ns`,
  `valid_at_ns`, or `expires_at_ns` has a test asserting the semantic
  instant.
- **Typed state machines.** Every `Readiness`, `LocalizationMode`,
  `PerceptionBackendHealth`, `MissionState`, `SafetyDecision`, etc., has
  a unit test per transition consumers branch on.
- **Typed reasons.** A typed reason enum gets per-variant transition
  tests.
- **Async cancellation safety.** Futures awaited inside `select!` or
  `join!` get drop-mid-await coverage.
- **Geometry math.** Known inputs/outputs including degenerate cases.
- **Resource budget declarations.** Each runtime's declared budget is
  reachable from a const in its API crate, with a parse test.
- **Calibration application.** A runtime that consumes per-device
  calibration has a test proving outputs differ when the overlay is
  present.
- **Regression coverage** for every fixed bug.

## What to do, what not to do

**Do:**

- Update the owning crate's `README.md` when a public contract changes.
- Add the Webots scenario gate when a runtime behavior changes.
- Write decode-error tests for new payloads, asserting topic + schema +
  version are in the error.
- Use Conventional Commits with the right prefix — `release-plz` reads them.
- Sign off commits per DCO.

**Don't:**

- Don't introduce a new `phoxal-*-api` crate without a clear owning runtime.
- Don't synthesize per-runtime config files.
- Don't add a runtime descriptor protocol.
- Don't add platform-runtime opt-out semantics.
- Don't open a live `Bus` outside the scenario carve-out in tests.
- Don't mix unrelated changes in one PR.
- Don't add backwards-compat shims for in-flight v0.x contracts.

## How to validate

- **Schema layer.** Each touched schema crate has parse + round-trip +
  `deny_unknown_fields` tests.
- **Contract layer.** Each touched `phoxal-*-api` crate has its drift /
  decode / actionable-error tests.
- **Scenario layer.** The relevant Webots gate from the framework's
  scenario catalog runs green. New behavior lands with its gate.
- **Clean-room flow.** From a clean machine with `phoxal-cli` installed:
  `phoxal-cli doctor --fix` → `phoxal-cli validate` → `phoxal-cli simulate <world>`.
  Critical changes require this to still pass.

## Visual verification

When the change is UI-, Rerun-, or Webots-rendering-related, the owning
repo's `AGENTS.md` will point at its visual-verification runbook (e.g.
`MACOS_VISUAL_VERIFICATION.md`, `RERUN_WEBOTS_VISUAL_VERIFICATION.md`).
Improvements to a runbook go through a normal reviewed commit, not an
in-place self-edit at the end of a run.

## Repo-local rules win for their repo

This guide is org-level. Each repo carries its own `AGENTS.md` for
implementation specifics (test fixtures, build hooks, local CI invocations,
visual runbooks). When a repo-local rule conflicts with this guide for that
repo's specifics, the repo-local rule wins.
