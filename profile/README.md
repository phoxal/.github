<div align="center">

<img src="https://phoxal.com/assets/phoxal-logo-vertical-transparent.png" alt="PHOXAL" width="220" />

### Rust-first robotics infrastructure for manifest-driven robots.

[![Status](https://img.shields.io/badge/Status-Pre--alpha-7fe7e3?style=flat-square&labelColor=06080b)](https://phoxal.com)
[![License](https://img.shields.io/badge/License-AGPL--3.0--only-87abc4?style=flat-square&labelColor=06080b)](https://www.gnu.org/licenses/agpl-3.0.html)
[![Commercial](https://img.shields.io/badge/Commercial-Available-4ccbc5?style=flat-square&labelColor=06080b)](https://phoxal.com)
[![Website](https://img.shields.io/badge/phoxal.com-7fe7e3?style=flat-square&labelColor=06080b)](https://phoxal.com)

</div>

---

> **Pre-alpha:** PHOXAL is being rewritten in public. APIs, catalogs, generated bundles, and runtime contracts may break loudly while the architecture converges. That is intentional: incoherent robot graphs should fail visibly, not drift silently.

PHOXAL is a **Rust-first, manifest-driven robotics framework** for teams that want the same model to drive validation, Webots simulation, deployment, and real robot execution.

It is not a ROS clone. The framework is built around authored robot and component contracts, typed Zenoh communication, isolated runtime and driver processes, logical simulation time, generated artifact catalogs, and operator-facing observability.

The first operational target is **sidewalk and street-maintenance autonomy**. The framework is general enough for mobile robots, but its design is forced by a real deployment goal: robots that can be modeled, simulated, observed, deployed, and fixed without hidden state.

## What PHOXAL optimizes for

| Principle | What it means |
|---|---|
| **Manifests are authoritative** | `robot.yaml`, `structure.urdf`, and component manifests describe the robot. Runtime state is not discovered implicitly. |
| **Contracts are typed** | Topics, queries, and commands are Rust contracts carried over Zenoh, not ad-hoc payloads on string channels. |
| **Simulation is first-class** | Webots simulation uses the same framework-owned contracts and a logical-time execution model. |
| **Processes are isolated** | Platform services, hardware drivers, user services, tools, and simulators are separate participants joined by the bus. |
| **Breaks are loud** | Pre-alpha API and catalog changes should fail clearly during resolution, validation, or decode, not degrade silently. |

## Where to start

| You want to … | Go here |
|---|---|
| **Read the framework code** | [`phoxal/framework`](https://github.com/phoxal/framework) — core crates plus official service, component, tool, and simulator binaries |
| **Use the consumer CLI** | [`phoxal/phoxal-cli`](https://github.com/phoxal/phoxal-cli) — validates manifests, resolves artifacts, runs simulation, and orchestrates local execution |
| **Inspect operator tooling** | [`phoxal/operator`](https://github.com/phoxal/operator) — host-side observability, currently centered on the Rerun proxy |
| **Try the public robot sandbox** | [`phoxal/robot-rover`](https://github.com/phoxal/robot-rover) — open rover manifest; migration to the current `robot/v0` grammar is still pending |
| **Understand org policy and CI** | [`phoxal/.github`](https://github.com/phoxal/.github) — this profile, community-health files, and reusable Rust CI/release workflows |

## Platform shape

```text
┌──────────────────────────────────────────────────────────────────────┐
│                            phoxal-cli                                │
│   check · validate · simulate · run · deploy · doctor · catalog       │
└──────────────────────────────────────────────────────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────────────────────────────────────┐
│                          phoxal/framework                            │
│                                                                      │
│  phoxal       authoring/runtime/model utilities                      │
│  phoxal-api   generation-qualified contracts: phoxal_api::y2026_1    │
│  phoxal-bus   typed Zenoh ABI floor                                  │
│  phoxal-macros service/driver/tool/simulator authoring macros        │
│                                                                      │
│  service/<name> · component/<name> · tool/<name> · simulator/<name>   │
│  official platform binaries, component packages, joypad, Webots       │
└──────────────────────────────────────────────────────────────────────┘
          ▲                              │
          │                              ▼
┌───────────────────────┐      ┌──────────────────────────────────────┐
│ Robot projects         │      │ phoxal/operator                      │
│ robot.yaml             │      │ host-side observability              │
│ structure.urdf         │      │ telemetry · logs · Rerun surfaces    │
│ services/              │      └──────────────────────────────────────┘
│ worlds/                │
└───────────────────────┘
```

## Status

PHOXAL is **pre-alpha and pre-1.0**. The public story is stable enough to present; the implementation is intentionally still moving.

Contracts live in generation-qualified modules such as `phoxal_api::y2026_1::<domain>`. Breaking changes should mint a new generation or catalog entry instead of quietly mutating an existing contract. Expect breakage while the framework converges.

## Conventions

- **License:** AGPL-3.0-only across public repos, with a commercial path for downstream products that cannot accept AGPL source-disclosure obligations.
- **Contributing:** DCO sign-off on every commit; Conventional Commits for messages.
- **Release model:** The framework publishes coherent artifact catalogs so the CLI can resolve a robot graph explicitly before launch.
- **Architecture stance:** No ROS-compat shim, no hidden runtime descriptor protocol, no freeform runtime graph as the default.

## Connect

- 🌐 [phoxal.com](https://phoxal.com)
- 💼 [LinkedIn](https://www.linkedin.com/company/phoxal)
- 💬 Issues + discussions on each individual repo

<sub>© PHOXAL · Built in the open.</sub>
