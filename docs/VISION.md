# Vision

Phoxal is a Rust framework for building production-oriented mobile robots that
ship — not a robotics research sandbox and not a ROS clone.

## What we're building

A coherent, manifest-driven robotics framework where a developer:

1. authors a robot in `robot.yaml` + `structure.urdf` + mounted components,
2. validates it,
3. simulates it in Webots,
4. observes it through operator tools,
5. and deploys it to a real device

…using the **same conventions, the same typed bus, and the same runtime set** end
to end. No protocol drift between simulation and deployment. No second mental model
for "production."

## Why

Existing open robotics stacks optimize for research flexibility — freeform runtime
graphs, dynamic discovery, schema-on-the-wire, ROS-shaped compatibility layers. That
flexibility is the wrong default when the goal is to actually ship a single robot
into a single operational environment, again and again, reliably.

Phoxal optimizes for the *shipping* side instead. It refuses to be generic where
opinionation makes the system easier to reason about: a complete platform runtime
set, typed bus contracts, a static catalog the CLI compiles against, manifests
parsed once and consumed everywhere.

## Principles

- **Explicit contracts over hidden state.** Every cross-runtime payload, query, or
  command lives in a published `phoxal-*-api` crate. If it isn't typed, it isn't
  a contract.
- **Manifests are authoritative.** `robot.yaml` + `structure.urdf` +
  `components/*/component.yaml` are the single source of truth for what a robot is.
  Runtimes parse them directly; nothing is synthesized into a side-channel
  descriptor.
- **Simulation-first validation.** Every contract has a Webots scenario or a
  headless validation gate. New behavior lands with its gate.
- **Strongly typed bus.** Zenoh under the hood, but every topic / query / command
  is bound to a Rust type. Decode errors fail loud with topic + schema + version.
- **Deterministic logical simulation time.** Sim is a clock the framework drives,
  not wall-time guesswork.
- **Coherent platform runtime set.** The mandatory runtimes ship together at one
  version; the CLI knows the topology statically. No per-runtime discovery
  protocol, no compatibility manifest, no descriptor preflight.
- **Runtime isolation.** Platform + user runtimes run in containers; hardware
  drivers and operator tools run native on the host. The bus is the only shared
  surface.
- **Repository ownership boundaries.** Each repo owns one concern (see
  [REPOSITORY_MAP.md](REPOSITORY_MAP.md)). Cross-repo coupling goes through
  published API crates, never through paths.
- **AGPL-3.0-only, with a commercial path.** Strong copyleft for the OSS suite;
  Phoxal offers a commercial license for downstream products that can't accept
  AGPL.

## What Phoxal intentionally avoids

- **No ROS-compat shim.** No `rosbridge`, no `tf2` parity, no `geometry_msgs`
  imitation. ROS interop, if it ever happens, is a separate concern owned by the
  consumer.
- **No freeform runtime graphs.** The mandatory platform runtime set is the
  framework's offer. Users *add* user-authored runtimes; they don't remove or
  rewire the platform ones.
- **No compatibility manifest, no runtime descriptor protocol, no
  `--phoxal-describe`.** Bus contracts are statically known to the CLI;
  mismatches fail at decode time with actionable errors.
- **No generated per-runtime config files.** Runtimes read `robot.yaml` and the
  resolved component definitions directly via the public schema crates.
- **No "platform fork to add a runtime."** User runtimes are additive, authored
  locally, built locally each `simulate`.

## Status

Pre-1.0, building in public. The shape above is settled; the APIs under each
`phoxal-*-api` crate are evolving inside `pub mod v1`. Breaking changes are
loud, not quiet.

For the current cross-repo state, see [REPOSITORY_MAP.md](REPOSITORY_MAP.md).
For the target architecture in depth, see [BLUEPRINT.md](BLUEPRINT.md).
