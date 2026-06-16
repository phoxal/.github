<div align="center">

<img src="https://phoxal.com/assets/phoxal-logo-vertical-transparent.png" alt="PHOXAL" width="220" />

### Open robotics infrastructure for autonomous urban operations.

[![Status](https://img.shields.io/badge/Status-Active%20development-7fe7e3?style=flat-square&labelColor=06080b)](https://phoxal.com)
[![License](https://img.shields.io/badge/License-AGPL--3.0--only-87abc4?style=flat-square&labelColor=06080b)](https://www.gnu.org/licenses/agpl-3.0.html)
[![Commercial](https://img.shields.io/badge/Commercial-Available-4ccbc5?style=flat-square&labelColor=06080b)](https://phoxal.com)
[![Website](https://img.shields.io/badge/phoxal.com-7fe7e3?style=flat-square&labelColor=06080b)](https://phoxal.com)

</div>

---

PHOXAL is a Rust-first robotics platform: **typed runtimes, simulation-first workflows, containerized service orchestration, and operator-facing observability** — released as a coherent set of open-source crates and tools. Built to take robots from a Webots scene to real operational environments without abandoning the same conventions along the way.

The first operational target is **sidewalk and street-maintenance autonomy**. The platform itself is general — it just refuses to be generic where opinionation makes the system easier to reason about.

## Where to start

| You want to … | Go here |
|---|---|
| **Drive a robot in simulation** | [`phoxal/phoxal-cli`](https://github.com/phoxal/phoxal-cli) — the consumer CLI |
| **Read the framework code** | [`phoxal/framework`](https://github.com/phoxal/framework) — engine + every platform runtime, one workspace |
| **Author a custom runtime** | The single published [`phoxal`](https://github.com/phoxal/framework) crate (`phoxal::{bus, model, spatial, api, runtime, scenario}`) |
| **Browse simulator integrations** | [`phoxal/framework`](https://github.com/phoxal/framework) — Webots controller + supervisor under `simulator/webots/` (wire contracts in `phoxal::api::simulation`) |
| **See operator tools** | [`phoxal/operator`](https://github.com/phoxal/operator) (observability); joypad teleop ships from [`phoxal/framework`](https://github.com/phoxal/framework) |
| **Pick a reference robot** | [`phoxal/robot-rover`](https://github.com/phoxal/robot-rover) — open exploratory rover manifest |
| **Hardware catalog** | [`phoxal/component-*`](https://github.com/orgs/phoxal/repositories?q=component) — driver + URDF + manifest per part |

## Platform shape

```
┌─────────────────────────────────────────────────────────────────┐
│                          phoxal-cli                             │
│      validate · simulate · doctor · create (robot.yaml)         │
└─────────────────────────────────────────────────────────────────┘
        │                                              │
        ▼                                              │
┌─────────────────────────┐                            │
│   phoxal/framework      │                            │
│  one `phoxal` crate     │                            │
│  + every platform       │                            │
│  runtime               ─┼─────┐                      │
│  + Webots ctrl/super-   │     │                      │
│  visor (simulator/)     │     │                      │
│  + joypad teleop        │     │                      ▼
│  (tools/joypad/)        │     │        ┌──────────────────────────┐
│  (router, asset,        │     └─────▶  │   phoxal/operator        │
│   drive, localize,      │              │  host-side observability │
│   safety, mission, …)   │              └──────────────────────────┘
└─────────────────────────┘
        ▲
        │
┌─────────────────────────┐
│  phoxal/component-*     │   per-hardware catalog —
│   driver + URDF +       │   referenced by robot.yaml
│   simulation manifest   │   via git + tag
└─────────────────────────┘
        ▲
        │
┌─────────────────────────┐
│  Robot manifests        │   robot.yaml + structure.urdf +
│   robot-rover (public)  │   user runtimes + sim worlds +
│   robot-v1 (private)    │   scenarios
└─────────────────────────┘
```

## Status

Active development, pre-1.0. The structure above is stable; the wire APIs under `phoxal::api` are evolving under `v1`. Expect changes; expect them to be loud.

## Documentation

Org-wide policy lives in this repo:

- [SECURITY.md](../SECURITY.md) — coordinated vulnerability disclosure
- [LICENSE_POLICY.md](../LICENSE_POLICY.md) — AGPL-3.0-only across public repos, with a commercial option

Architecture, vision, the full repository map, and the cross-repo development model live in the `phoxal/organization` control repo.

## Conventions

- **License**: AGPL-3.0-only across every public repository. A commercial license is available for downstream products that cannot meet the AGPL source-disclosure obligations — see each repo's `COMMERCIAL.md` or reach out via [phoxal.com](https://phoxal.com).
- **Contributing**: DCO sign-off on every commit; Conventional Commits for messages. Details in each repo's `CONTRIBUTING.md`.
- **Coherent release**: `phoxal/framework` releases the `phoxal` crate, every runtime image, the Webots simulator binaries, and the joypad tool together at one version. The CLI gates resolution by that runtime-set version.

## Connect

- 🌐 [phoxal.com](https://phoxal.com)
- 💼 [LinkedIn](https://www.linkedin.com/company/phoxal)
- 💬 Issues + discussions on each individual repo

<sub>© PHOXAL · Built in the open.</sub>
