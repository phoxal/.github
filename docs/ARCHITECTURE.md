# Architecture

This document describes the system model that Phoxal converges to. For
principles see [VISION.md](VISION.md). For the cross-repo layout see
[REPOSITORY_MAP.md](REPOSITORY_MAP.md). For the autonomy target see
[BLUEPRINT.md](BLUEPRINT.md).

## System model

```
robot.yaml + structure.urdf + components/* (authored)
        │
        ▼
phoxal-cli validate         (typed schemas, cross-file rules)
        │
        ▼
phoxal-cli simulate         (resolve digests + SHAs, assemble .phoxal/run/)
        │
        ▼
docker-compose: router + every platform runtime + user runtimes
+ native processes: drivers, Webots, operator tools
        │
        ▼
typed Zenoh bus (every topic/query/command bound to a phoxal-*-api crate)
        │
        ▼
operator app / rerun-proxy / joypad — observability + teleop
```

Same pipeline drives deployment — only the simulator and a subset of native
host tools drop out, replaced by real hardware drivers and a production router
endpoint.

## Authoring model

A robot is defined by three kinds of files:

- **`robot.yaml`** at the project root. The single project marker and the
  authored manifest: identity, structure path, mounted component instances,
  motion configuration, runtime selection (mandatory platform set version,
  optional per-runtime overrides), user runtime list, optional production
  network endpoints.
- **`structure.urdf`** — the kinematic tree. URDF, with Phoxal conventions on
  link naming used by the platform runtimes that consume structure (`drive`,
  `frame`, `localize`, `safety`).
- **`components/<name>/`** — one directory per component. Each carries its own
  `component.yaml` (capabilities, GTIN, driver requirements), `structure.urdf`
  (the component's physical shape), `simulation.yaml` (sim-side hints), and
  optionally a `driver/` Cargo crate that implements the hardware-facing side.

Components are consumed in two ways: **catalog components** are referenced from
`robot.yaml`'s `components.sources` as `git + tag` (resolved to a commit SHA in
`phoxal.lock`); **custom components** live as `path:` entries pointing at a
robot-local `components/<name>/` directory.

There is no `phoxal.yaml`, no separate runtime descriptor, no synthetic
side-channel manifest. `robot.yaml` is everything the user authors.

## Validation model

`phoxal-cli validate` runs three layers:

1. **Schema layer.** Each authored file parses through its public schema crate:
   `phoxal-core-robot`, `phoxal-core-structure`, `phoxal-core-component`. Schema
   errors are typed and point at the offending file + line.
2. **Cross-file layer.** Each `components.instances.<name>.component` resolves
   to a `components.sources` entry; component capabilities tagged
   `localization` satisfy the eligibility of the resolved localize backend;
   motion configuration's `left_actuators` / `right_actuators` references live
   inside `structure.urdf`; etc.
3. **Scenario layer.** `phoxal-cli validate scenario` runs Webots-backed gates
   from the framework's catalog (e.g. `p2-localization-rgbd-inertial-orb-slam3`,
   `p3-safety-range-sensor-staleness`). Each platform runtime owns a phase gate;
   the gates run in order and fail loud.

## Runtime graph model

Two distinct runtime layers, with deliberately different rules:

**Platform runtimes** — the always-present substrate every Phoxal robot runs.
The complete set is determined by the `phoxal/framework` workspace version the
robot pins. The user can *version* or *override* a platform runtime; the user
cannot *remove*, *skip*, or *replace* one. The complete set is the framework's
offer.

**User runtimes** — robot-specific services the user authors under
`runtimes/<name>/`. Additive only: they extend the robot with custom behavior
(mission logic, inspection policies, custom control loops). They cannot shadow
a platform runtime's name. They are built locally each `simulate`.

The runtime graph is **not** a runtime artifact. The CLI knows the full
platform runtime topology statically via a `PlatformRuntimeCatalog` baked into
its source code, indexed by runtime-set version. No graph preflight, no
descriptor introspection, no `--phoxal-describe`.

## Communication model

All cross-process communication goes over a typed Zenoh bus.

- Every topic, query, and command payload is defined in a public `phoxal-*-api`
  crate (`phoxal-bus`, `phoxal-api-component`, one
  `phoxal-runtime-<name>-api` per platform runtime, `phoxal-simulator-api`).
- A subscriber that decodes a payload it doesn't understand fails loud — the
  error carries the topic, expected schema + module + version, payload type, and
  the runtime / tool reporting it. This is the framework's stand-in for a
  compatibility manifest: contracts are statically known to the CLI; mismatches
  surface at decode time, debuggably.
- API contract versions evolve inside `pub mod vN` modules of each `-api`
  crate. Additive changes go in the existing `vN`; breaking changes mint a
  `v(N+1)`. Removal happens at a major crate bump.

## Simulation model

`phoxal-cli simulate <world>` builds a two-layer stack:

| Layer | Runs in docker? | What lives there |
|---|---|---|
| Robot router | Yes — upstream `eclipse/zenoh` image, pinned by digest | One per robot compose project |
| Platform runtimes | Yes — pulled from GHCR by locked digest | asset, presence, frame, safety, drive, localize, map, mission, plan, perception, … |
| User runtimes | Yes — built locally, transient cache | `<robot>/runtimes/<name>/` |
| Component drivers | **No — native** | `cargo build && cargo run` of each driver crate the robot's instances reference |
| Webots integration | **No — native** | downloaded host binaries from `~/.phoxal/cache/` |
| Operator tools | **No — native** | downloaded host binaries from `~/.phoxal/cache/` |

The two layers join at the bus. The robot's compose `router` service federates
with a host-side `phoxal-local-zenoh` container across the external
`phoxal-link` bridge network; native processes connect to
`127.0.0.1:7447`. Cross-platform behavior is identical on Linux and macOS
Docker Desktop.

Simulation logical time is driven by the Webots supervisor. Platform runtimes
that consume time (`frame`, `localize`, `mission`, etc.) run against engine
clock primitives, not wall time, so scenario replays are deterministic.

## Deployment model

`phoxal-cli build` and `phoxal-cli emit compose` / `emit balena-release` /
`emit systemd` / `emit k8s` are the v1+ deployment surface (tracked under
[phoxal/robot-framework#815](https://github.com/phoxal/robot-framework/issues/815)).

The framework's deployment posture: **facilitate, don't decide.** The CLI
produces pinned artifacts — multi-arch images on GHCR by digest, generated
compose / balena release manifests referencing those digests; the user wires
them into their own deploy pipeline. Phoxal does not own deploy topology
decisions (cgroups, systemd unit shape, k8s scheduling); those belong to the
target.

`phoxal.lock` is what makes deployment reproducible: the locked image digests,
component SHAs, and tool hashes flow from `simulate` straight into the
artifacts `build` produces.

## Observability model

- **rerun-proxy** — host-side raw camera/depth visualizer. Subscribes to
  capability streams over the bus and renders them in Rerun.
- **operator** — Phoxal's main host-side UI for status, logs, manual control,
  and scenario runs.
- **joypad** — host-side joystick teleop. Publishes `ManualCommand` payloads
  through `phoxal-runtime-drive-api`.

Every operator tool consumes the same `phoxal-*-api` crates the runtimes
publish. There are no operator-specific contracts (no `phoxal-joypad-api`, no
`phoxal-rerun-proxy-api`); the tools are bus consumers, not bus owners.

## What belongs where

| Concern | Owner |
|---|---|
| Engine primitives (`Runtime` trait, bootstrap, runtime context, logical clock, presence, bus integration) | `phoxal/framework` engine crates |
| Bus transport + typed pubsub | `phoxal/framework` (`phoxal-bus`) |
| Authored schemas (`robot.yaml`, `structure.urdf`, `component.yaml`) | `phoxal/framework` schema crates |
| Per-platform-runtime contracts | `phoxal/framework` runtime-`-api` crates |
| Platform runtime behavior | `phoxal/framework` runtime binaries |
| Resolver, `phoxal.lock`, command implementations, native-process orchestration | `phoxal/phoxal-cli` |
| Simulator-side protocol + Webots controller/supervisor | `phoxal/simulator` |
| Hardware integration (catalog components) | `phoxal/component-<name>` |
| Robot manifest + custom components + user runtimes + worlds + scenarios | user-owned robot repo (e.g. `phoxal/robot-rover`) |
| Operator UI / teleop / debug visualizers | `phoxal/operator`, `phoxal/joypad` |

When in doubt, the owning repo is the one whose contracts the change crosses.
A change that modifies a `phoxal-runtime-<name>-api` payload lands in
`phoxal/framework`; consumers update to the new contract from crates.io.
