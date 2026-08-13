<div align="center">

<img src="https://phoxal.com/assets/phoxal-logo-vertical-transparent.png" alt="PHOXAL" width="220" />

### Robots that improve from experience—without giving up explicit control.

[![Status](https://img.shields.io/badge/Status-Pre--alpha-7fe7e3?style=flat-square&labelColor=06080b)](https://phoxal.com)
[![License](https://img.shields.io/badge/License-AGPL--3.0--only-87abc4?style=flat-square&labelColor=06080b)](https://www.gnu.org/licenses/agpl-3.0.html)
[![Commercial](https://img.shields.io/badge/Commercial-Available-4ccbc5?style=flat-square&labelColor=06080b)](https://phoxal.com)
[![Website](https://img.shields.io/badge/phoxal.com-7fe7e3?style=flat-square&labelColor=06080b)](https://phoxal.com)

</div>

---

> **Pre-alpha:** PHOXAL is being built in public. API and runtime contracts may break loudly while the framework converges. That is intentional: incoherent robot graphs should fail visibly, not drift silently.

**Phoxal Framework** is an opinionated Rust-first robotics suite that carries one explicit model from authoring through deterministic validation, Webots simulation, native execution, observability, and rollback-safe deployment.

It is not a ROS clone and its goal is not to put an LLM in charge of a robot. The framework is built around authored robot and component models, a closed framework-owned semantic API, isolated native processes, logical simulation time, validated runtime bundles, and typed evidence from every run.

## North Star

> **Phoxal Framework should let robots improve from experience without giving up explicit authority, deterministic validation, observable state, simulation parity, repeatability, or rollback-safe deployment.**

The improvement loop is explicit: execution produces typed experience and provenance; training or derivation happens outside the robot runtime; deterministic scenarios accept or reject a candidate; and only a passing immutable policy or model enters the next validated bundle. Deployed processes never silently replace production behavior.

```text
execute → record + feedback → train / derive → evaluate → promote → next validated bundle
   ▲                                                               │
   └──────────────────────── fallback + rollback ───────────────────┘
```

## What Phoxal Framework optimizes for

| Principle | What it means |
|---|---|
| **Authority stays explicit** | The root brain owns application policy; framework domain authorities and safety boundaries remain enforceable. |
| **The semantic API is closed** | Robot processes use framework-owned typed contracts over Zenoh, not project-private endpoints or arbitrary payloads. |
| **Simulation and hardware agree** | Webots and native execution use the same semantic graph, authorities, and logical-time rules. |
| **Evidence is typed** | Recordings preserve state, timing, provenance, outcomes, and the exact artifact that made a decision. |
| **Promotion is deliberate** | Candidates are evaluated deterministically, then enter the ordinary immutable bundle and rollback path. |

## Where to start

| You want to … | Go here |
|---|---|
| **Read the framework code** | [`phoxal/framework`](https://github.com/phoxal/framework) — the closed semantic API, model, bundle, bus, authoring facade, and official participants |
| **Use the CLI and supervisor** | [`phoxal/phoxal-cli`](https://github.com/phoxal/phoxal-cli) — validates, builds, simulates, deploys, supervises, and attaches |
| **Try an unrelated robot** | [`phoxal/robot-rover`](https://github.com/phoxal/robot-rover) — the public sandbox and genericity check |
| **Read the product story** | [phoxal.com](https://phoxal.com) — the public Phoxal Framework direction and project status |
| **Understand org policy and CI** | [`phoxal/.github`](https://github.com/phoxal/.github) — this profile, community-health files, and reusable Rust CI/release workflows |

## Minimum Working Framework

```text
author robot + components + brain
    → validate deterministically
    → build an inspectable runtime bundle
    → run the same semantic graph in Webots and on hardware
    → deploy, attach, inspect, manually drive where supported, stop, and restart
    → collect enough evidence to reproduce a failure
```

That end-to-end path is the v1 usability floor, not the reason to call the framework 1.0. Reference applications qualify the framework; application-specific missions and vocabulary do not define it.

## Status

PHOXAL is **pre-alpha and pre-1.0**. The direction is explicit; the implementation is intentionally still moving. Expect breaking API and runtime changes while the framework converges, and expect them to fail loudly rather than drift silently.

## Conventions

- **License:** AGPL-3.0-only across public repos, with a commercial path for downstream products that cannot accept AGPL source-disclosure obligations.
- **Contributing:** DCO sign-off on every commit; Conventional Commits for messages.
- **Release model:** The framework ships one coherent versioned train; the CLI validates compatibility before launch.
- **Architecture stance:** No ROS-compat shim, no hidden runtime descriptor protocol, no freeform runtime graph as the default.

## Connect

- 🌐 [phoxal.com](https://phoxal.com)
- 💼 [LinkedIn](https://www.linkedin.com/company/phoxal)
- 💬 Issues + discussions on each individual repo

<sub>© PHOXAL · Built in the open.</sub>
