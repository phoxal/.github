# Repository Map

The Phoxal framework spans several focused repositories under
[github.com/phoxal](https://github.com/phoxal). Each repo owns one concern;
cross-repo coupling goes through published API crates on crates.io and pinned
container images on GHCR.

This document reflects the **current public-repo layout**. The deeper engine /
runtimes split tracked in `phoxal/robot-framework#823` is in flight; until it
lands, the framework workspace below contains both the engine library crates
and the platform runtime binaries.

## Public repositories

| Repo | Role | Publishes |
|---|---|---|
| [`phoxal/framework`](https://github.com/phoxal/framework) | Engine library crates (`phoxal-engine`, `phoxal-bus`, schema crates) + every platform runtime binary, one workspace. | Engine + bus + schema crates → crates.io; one `phoxal-runtime-<name>-api` crate per platform runtime → crates.io; one multi-arch image per platform runtime → GHCR. |
| [`phoxal/phoxal-cli`](https://github.com/phoxal/phoxal-cli) | Consumer CLI. Resolver, `robot.yaml` discovery, `phoxal.lock`, `validate` / `simulate` / `doctor` / `create`. Consumes the framework schema crates. | Per-host-platform prebuilt binaries → GitHub Releases; brew tap; `cargo install phoxal-cli`. |
| [`phoxal/simulator`](https://github.com/phoxal/simulator) | Webots controller + supervisor + the `phoxal-simulator-api` crate. Other simulators add as sibling subdirs. | `phoxal-simulator-api` → crates.io; per-host-platform Webots binaries → GitHub Releases. |
| [`phoxal/operator`](https://github.com/phoxal/operator) | Host-side observability and teleop UI. Consumes runtime API crates. | Per-host-platform binaries → GitHub Releases. |
| [`phoxal/joypad`](https://github.com/phoxal/joypad) | Host-side joystick teleop tool. Consumes runtime API crates. | Per-host-platform binaries → GitHub Releases. |
| [`phoxal/component-<name>`](https://github.com/orgs/phoxal/repositories?q=component) | One repo per first-party catalog component (`bno085`, `ddsm115`, `oak_d_lite`, `vl53l1x`, `zed_f9p`, …). Each carries `component.yaml` (GTIN-13), `structure.urdf`, `simulation.yaml`, and optionally a `driver/` Cargo crate. | Git tags; referenced from `robot.yaml`'s `components.sources` as `git + tag`, resolved to a commit SHA in `phoxal.lock`. |
| [`phoxal/robot-rover`](https://github.com/phoxal/robot-rover) | Open exploratory robot manifest used as a reference. | Not published — consumed as a user robot project. |
| [`phoxal/.github`](https://github.com/phoxal/.github) | This repo. Organization profile + cross-repo docs + agent prompts + governance. | Not published. |

## Private repositories

| Repo | Role |
|---|---|
| `phoxal/robot-framework` | Control plane for the multi-repo split. Holds cross-repo planning ([Phoxal Kanban](https://github.com/orgs/phoxal/projects/1)), reconciliation notes, and recovery scripts. **No code.** |
| `phoxal/robot-v1` | v1 reference robot (manifests, custom components, user runtimes, scenarios). |

## How a robot project consumes the framework

A user robot repository is **not** in this org — it lives under the user's own
GitHub account or org and follows the layout below:

```
my-robot/
  robot.yaml                    # project root marker + authored manifest
  phoxal.lock                   # exact resolved digests + git SHAs (committed)
  structure.urdf
  components/                   # robot-local custom components
    <name>/
      component.yaml
      structure.urdf
      simulation.yaml
      driver/                   # optional
  runtimes/                     # user-authored additive runtimes
    <name>/
      Cargo.toml
      src/main.rs
  worlds/                       # robot-specific Webots worlds
  scenarios/                    # user-authored validation scenarios
```

`phoxal-cli simulate <world>` resolves `phoxal_runtimes.version` from
`robot.yaml`, pulls every platform runtime image from GHCR by locked digest,
builds the user runtimes locally, fetches each `components.sources.<name>` from
its git origin, and brings the stack up in Webots.

## Dependency direction

```
phoxal/framework ──► (crates.io: engine, bus, schemas, runtime-*-api)
phoxal/simulator ──► (crates.io: simulator-api; releases: webots binaries)
phoxal/component-* ──► (git tags consumed by robot.yaml)

phoxal/phoxal-cli ──┐
phoxal/operator   ──┤ consume framework + simulator API crates from crates.io
phoxal/joypad     ──┘

robot projects (private) ──► consume phoxal-cli, framework + simulator API crates, component git refs
```

A change that modifies a cross-runtime contract lands in `phoxal/framework`
first, then propagates to consumers. See [DEVELOPMENT_MODEL.md](DEVELOPMENT_MODEL.md).

## Planned reshape

The framework workspace currently bundles engine library crates and platform
runtime binaries together. The tracked split (per
[phoxal/robot-framework#823](https://github.com/phoxal/robot-framework/issues/823))
separates these into:

- `phoxal/engine` — the engine library crates + schema crates (`phoxal-engine`, `phoxal-bus`, `phoxal-api-component`, `phoxal-core-robot`, `phoxal-core-structure`, `phoxal-core-component`).
- `phoxal/runtimes` — the platform runtime binaries + their `phoxal-runtime-<name>-api` crates, released as one coherent workspace version.

This map will be updated when those repos exist. The dependency direction does
not change.
