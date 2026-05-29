# Security Policy

## Reporting a vulnerability

**Do not** open a public GitHub issue for security vulnerabilities.

Report security issues privately via either:

1. **GitHub Security Advisories** — go to the affected repo →
   *Security* → *Report a vulnerability*. This is the preferred channel; it
   keeps the discussion private until coordinated disclosure.
2. **Email** — `security@phoxal.com`. Use this if you can't access the
   Security tab or if the vulnerability spans multiple repositories.

Please include:

- the affected repository (or "multiple — see body");
- a description of the vulnerability;
- a proof of concept if possible (one is appreciated, not required);
- the impact you've observed or can reasonably anticipate;
- any mitigations or workarounds you've identified.

We'll acknowledge receipt within a few business days. Phoxal is a small
team; please be patient if a deeper investigation takes longer than that.

## Scope

In scope:

- The code in any [`phoxal/*`](https://github.com/phoxal) repository.
- The published crates on crates.io under the `phoxal-*` prefix.
- The runtime images published to GHCR under `ghcr.io/phoxal/*`.
- The CLI binaries published to GitHub Releases under any Phoxal repo.

Out of scope (please report to the upstream maintainer instead):

- Vulnerabilities in upstream dependencies (e.g. `eclipse/zenoh`, third-party
  crates). Phoxal will track upstream advisories and bump.
- User-authored robot manifests, user runtimes, or custom components.
- Issues in user-deployed robots in operational environments — we can advise
  but we cannot remediate on a specific robot remotely.

## Coordinated disclosure

We prefer coordinated disclosure. If the issue is severe, please give us a
reasonable window (typically 30–90 days) to ship a fix before public
disclosure. We will credit you in the advisory if you'd like; we'll honor
"no credit" requests too.

## What we won't do

- We won't take legal action against good-faith security research.
- We won't penalize reporters for following this process.
- We won't ignore reports — if you don't hear back within a week, please
  resend or escalate.

## Supply chain

Trust hardening (image signing, component allowlists, reproducible builds)
is in progress and tracked under
[phoxal/robot-framework#822](https://github.com/phoxal/robot-framework/issues/822).
Until those land, the trust minimum is **`phoxal.lock` pins SHAs and
digests; deploy consumes the lock, never a mutable tag**.

If you spot a supply-chain risk that this minimum doesn't cover, file it
through the channels above.
