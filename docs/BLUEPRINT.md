# Blueprint

This is the target architecture for Phoxal's first major version. For
principles see [VISION.md](VISION.md); for system shape see
[ARCHITECTURE.md](ARCHITECTURE.md); for cross-repo layout see
[REPOSITORY_MAP.md](REPOSITORY_MAP.md).

## The v1 promise

> A validated ground vehicle can autonomously navigate a never-before-seen
> environment using framework-resolved runtime configuration, typed runtime
> contracts, simulation validation, and deployment artifacts derived from the
> authored robot model and structure.

The framework is **vehicle-class aware, not vehicle-class naive**. A robot is
autonomy-capable only if it validates against an explicit autonomy profile.
A profile defines required sensing, kinematics support, safety envelope,
localization capability, world-model capability, and scenario coverage.

The v1 reference profile is concrete:

- ground vehicle
- terrain-wheeled or skid/differential reference kinematics
- mostly planar navigation with terrain/slope/support awareness
- RGB-D or lidar plus IMU plus wheel/drive feedback
- optional near-field proximity (single-beam ToF / short-range rangefinder) for safety
- optional GNSS global-frame anchor for outdoor profiles
- optional onboard perception producing typed detections
- conservative autonomous speeds
- unknown-environment exploration and directed navigation
- Webots scenario validation before deployment, with hardware log-replay for
  conditions Webots cannot reproduce

The contracts support other profiles; v1 just commits to finishing this one.

## Authoring contract

The developer authors **robot and component facts only**:

- identity (`id`, `namespace`)
- mounted component instances + connections
- motion configuration (kinematic class, wheel/drive references)
- base `structure.urdf`
- component-local definitions (capabilities, GTIN, driver requirements)

The developer **does not** hand-author runtime dependency wiring for
localization, mapping, planning, safety, or driving.

The single source of truth is the authored set: `robot.yaml`,
`structure.urdf`, and the resolved `components/<name>/component.yaml` files.
Each runtime parses what it needs directly from the mounted robot view at
`/robot` — there is no synthetic per-runtime config file, no
`runtime.graph.yaml`, no `topics.manifest.yaml`, no
`autonomy.profile.yaml` consumed by runtimes.

## Resolution model

Each runtime process loads the source-shaped staged bundle:

```text
robot.yaml
structure.urdf
components/<name>/component.yaml
components/<name>/structure.urdf
components/<name>/simulation.yaml
```

It then calls the shared deterministic resolver from the public schema crates
(`phoxal-core-robot`, `phoxal-core-structure`, `phoxal-core-component`) and
extracts only its own typed slice. The resolver is shared, deterministic, and
tested. Each runtime owns its domain-specific resolved struct; the schema
crates stay clean of localization, map, mission, safety, or drive semantics.

`phoxal.lock` records exact non-Cargo artifact pins (platform runtime image
digests, component git SHAs, host-tool release asset hashes). It is committed
to the robot repo and consumed by `phoxal-cli` for reproducible
`simulate` / `build`. It is **not** runtime semantic input.

## Runtime model

Two distinct layers:

**Platform runtimes** — the always-present substrate. The complete set is
determined by the `phoxal/framework` workspace version the robot pins. The
user can *version* or *override* a platform runtime; the user cannot *remove*,
*skip*, or *replace* one.

**User runtimes** — robot-specific services the user authors under
`runtimes/<name>/`. Additive only. They extend the robot with custom behavior.
They cannot shadow a platform runtime's name. They are built locally each
`simulate`.

The complete platform set ships **together at one workspace version**. Each
release tags every runtime image at the same version and publishes every
`phoxal-runtime-<name>-api` crate at the same version. Internally each api
crate uses `pub mod vN` modules for gradual API evolution within a release.

The CLI carries a static `PlatformRuntimeCatalog` indexed by runtime-set
version: the complete set of mandatory names, GHCR image patterns, compose
service shape, required environment, and per-runtime defaults. When a new
runtime-set release adds or changes a mandatory runtime, the CLI must be
updated. This is the price of avoiding a compatibility manifest: the CLI owns
topology knowledge statically; mismatches surface at decode time.

## Platform runtime categories

Concrete names will evolve with the workspace; the categories below are
stable.

| Category | Purpose | Examples |
|---|---|---|
| **Transport** | Per-robot bus router (upstream `eclipse/zenoh` image, pinned by digest). No Phoxal binary; inserted by the CLI. | `router` |
| **System** | Asset distribution, presence, power, version reporting. | `asset`, `presence`, `power` |
| **Time + structure** | Joint state, time-indexed frame lookup, dead-reckoning. | `joint`, `frame`, `odometry` |
| **Perception + world** | Visual / visual-inertial localization, revisioned submap map, optional onboard detections. | `localize`, `map`, `perception` |
| **Behavior** | Safety envelope, mission state, plan, exploration, follow, drive command translation. | `safety`, `mission`, `plan`, `explore`, `follow`, `motion`, `drive` |
| **Operator delivery** | Codec-encoded camera streams for operator/web. (Rerun consumes raw driver products directly.) | `video` |

Each runtime publishes a typed contract through its own
`phoxal-runtime-<name>-api` crate. Topic shape, query semantics, command
shape, and any state machine consumers branch on (e.g. `LocalizationMode`,
`SafetyDecision`, `MissionState`) live in the api crate, not in the runtime
binary.

## Communication contract

- Every payload is bound to a Rust type defined in a public `phoxal-*-api`
  crate.
- A subscriber that decodes a payload it does not understand fails loud. The
  decode error carries: topic, expected schema crate + module + version,
  payload type, and the runtime / tool name reporting the error.
- Additive API changes go inside the existing `pub mod vN`. Breaking changes
  mint `pub mod v(N+1)`. An api crate can serve multiple `vN` simultaneously;
  removal happens at a major crate bump.

There is **no runtime descriptor protocol**, no `--phoxal-describe`, no OCI
label introspection, no graph preflight before docker-compose. CI scenarios
in `phoxal/framework` validate that all platform runtimes at a given workspace
version interoperate. The user's per-runtime overrides are at the user's risk.

## Revision convergence

Stateful runtimes (`localize`, `map`, `perception` …) expose **revision-pinned
state**: every product a downstream consumer reads carries the revision under
which it was produced. Consumers either accept the revision or reject the
message. On a localization-driven correction (loop closure, etc.) the upstream
runtime increments its revision and the downstream world model converges to
match.

This is the contract that lets autonomy survive recoveries without
silently mixing pre- and post-correction state.

## Simulation contract

`phoxal-cli simulate <world>` always brings up the **complete mandatory
platform runtime set**, plus every user runtime, with all platform runtimes
running against the same workspace version. The simulator drives logical
time; runtimes consume engine clock primitives, not wall time.

Drivers, Webots integration, and operator tools run as native processes on
the host; bus participants that ship as images run in docker. The two layers
join through the bus.

A Webots scenario asserts the full chain end to end: validation passes →
runtimes reach `Ready` → autonomy reaches `Tracking` → mission executes → the
world model converges around the revision boundary → no safety violation. A
scenario that does not exercise this chain is not a scenario.

## Validation contract

Three layers, all driven by `phoxal-cli`:

1. **Schema validation** — every authored file parses through its public
   schema crate. Type errors are surfaced with file + line.
2. **Cross-file validation** — component-instance references resolve, role
   tags satisfy backend eligibility, motion references live inside the
   structure, etc.
3. **Scenario validation** — Webots-backed gates from the framework's
   catalog, organized into phase tiers. A v1 robot is autonomy-capable only
   when its profile's scenario tiers are green.

Each platform runtime owns a phase gate. Adding a runtime adds a gate.
Changing a contract changes a gate. No silent contract drift.

## Deployment contract

`phoxal-cli build` produces durable, deployable robot images (multi-arch);
`phoxal-cli emit compose` / `emit balena-release` / … produce digest-pinned
manifests for the user's deploy pipeline. Reproducibility = `phoxal.lock`'s
exact pins.

The framework **facilitates, does not decide.** Deploy topology (cgroups,
systemd unit shape, k8s scheduling, balena supervisor wiring) belongs to the
deploy target, not to the framework. The `emit` commands hand the user pinned
artifacts; the user wires them into their own pipeline.

This v1+ surface is tracked under
[phoxal/.github#5](https://github.com/phoxal/.github/issues/5).

## Observability contract

- `runtime/<name>/state` — typed status every runtime publishes.
- `runtime/presence/summary` — operator-facing compact readiness.
- `runtime/asset/get` — query access to robot structure, model, component
  configs, and any staged assets.
- `runtime/video/stream/<id>` — codec-encoded camera previews for operator /
  web tools.
- Raw camera / depth / pointcloud capability topics drive `rerun-proxy`
  directly (no encode hop).

Operator tools (`phoxal/operator`, `phoxal/joypad`, `rerun-proxy`) consume
the same `phoxal-*-api` crates the runtimes publish. There are no
operator-specific contracts; tools are bus consumers, not bus owners.

## Versioning

Two distinct version axes that should not be conflated:

- **Cargo crate version** = the workspace release packaging. `phoxal-api-drive = "0.1.9"` selects the release.
- **Rust module version** (`pub mod vN`) = the wire contract. `use phoxal_runtime_drive_api::v1::*;` selects the protocol boundary.

The runtime workspace publishes every `-api` crate at the workspace version
on every release tag. Per-runtime independent semver tracks are explicitly
not a thing: coherence comes from the workspace release.

## License

AGPL-3.0-only (SPDX) across every public Phoxal repository. The `phoxal-cli
create` scaffolds default to AGPL-3.0-only for user runtime code and custom
components. A commercial license is available from Phoxal for downstream
products that cannot accept AGPL — see each repo's `COMMERCIAL.md` or
[phoxal.com](https://phoxal.com).

## Definition of done for autonomy v1

A robot that validates against the v1 autonomy profile can:

- boot from staged bundle artifacts
- publish presence and autonomy readiness
- provide time-indexed frame lookup
- localize into `Tracking`
- maintain revisioned localization state
- build a revisioned submap-based world model with a traversability view
- serve revision-pinned map products
- plan a kinematically feasible path
- follow the path under safety constraints
- recover within the resolved revision-convergence budget after loop closure
- explore an unseen environment with measurable coverage growth
- stop safely before obstacle / contact violations
- expose mission state, failure reasons, and debug products
- render behavior from explicit products only (no Rerun-side reconstruction)
- pass required Webots scenario tiers
- deploy to the chosen target using the same resolved views and lockfile

## Non-goals

- **No ROS-compat layer.** No `rosbridge`, no `tf2` parity, no
  `geometry_msgs` imitation.
- **No freeform runtime graph.** The mandatory platform set is the
  framework's offer.
- **No compatibility manifest, no runtime descriptor, no preflight
  introspection.** Bus contracts are statically known to the CLI.
- **No generated per-runtime config files.** Runtimes read `robot.yaml` and
  the resolved component definitions directly.
- **No bundle.descriptor.yaml.** Removed; `phoxal.lock` carries the
  reproducibility pins.
- **No per-runtime independent semver tracks.** The workspace release is
  the coherence unit.
- **No platform-runtime replacement by user runtimes.** User runtimes are
  additive only.

This blueprint is target architecture, not a cleanup plan. Where the
current implementation differs (legacy `map` as a fixed-extent occupancy
grid, planar-only `odometry`, deploy descriptor remnants), those are
placeholders being replaced, not an incremental base to extend.
