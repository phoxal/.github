# Development Model

How to contribute across the Phoxal repositories.

## The repos you'll work in

See [REPOSITORY_MAP.md](REPOSITORY_MAP.md) for the full picture. The short
version:

- [`phoxal/framework`](https://github.com/phoxal/framework) — engine library
  crates + platform runtime binaries.
- [`phoxal/phoxal-cli`](https://github.com/phoxal/phoxal-cli) — the consumer
  CLI.
- [`phoxal/simulator`](https://github.com/phoxal/simulator) — Webots
  controller + supervisor + `phoxal-simulator-api`.
- [`phoxal/operator`](https://github.com/phoxal/operator) — host-side UI.
- [`phoxal/joypad`](https://github.com/phoxal/joypad) — joystick teleop.
- [`phoxal/component-<name>`](https://github.com/orgs/phoxal/repositories?q=component) — first-party catalog components.
- [`phoxal/robot-rover`](https://github.com/phoxal/robot-rover) — open
  reference robot.

## Where does my change belong?

Use this matrix:

| Change touches… | Owning repo |
|---|---|
| `Runtime` trait, bus integration, schema crates, runtime context, logical clock | `phoxal/framework` |
| A platform-runtime contract payload (anything published on a `phoxal-runtime-<name>-api` topic) | `phoxal/framework` |
| Platform runtime behavior | `phoxal/framework` |
| Resolver, `phoxal.lock`, `validate` / `simulate` / `doctor` / `create` commands, native-process orchestration | `phoxal/phoxal-cli` |
| Simulator-side protocol or the Webots controller/supervisor | `phoxal/simulator` |
| A new hardware integration (catalog component) | `phoxal/component-<name>` (new repo per component) |
| Operator UI, debug visualizers, host-side teleop | `phoxal/operator` or `phoxal/joypad` |
| Your robot's manifest, custom components, user runtimes | your robot repo (not in the `phoxal/` org) |

If the change crosses owners, **the owner repo of the contract goes first**,
then consumers update from crates.io / GHCR. A change that modifies a
`phoxal-runtime-<name>-api` payload lands in `phoxal/framework`; the CLI,
simulator, operator, and user-runtime consumers follow.

## Cross-repo contract changes

The rule: **API crate first, then consumers.**

```
1. Land the typed payload change in phoxal/framework on a feature branch.
2. Bump the runtime-set workspace version (or add a new `pub mod vN`).
3. Tag + release the framework workspace (publishes all -api crates + images).
4. Update each consumer to the new crate version.
5. Drop the temporary git-based [patch.*] block if you used one locally.
```

Skipping step 3 (releasing the api crates) is the most common source of
"works on my machine" cross-repo breakage. Use sibling checkouts with
`[patch.*]` blocks while iterating, but release before merging consumers.

## Local sibling checkouts

The recommended layout for working across more than one repo at a time:

```
~/phoxal/
  framework/
  phoxal-cli/
  simulator/
  operator/
  joypad/
  component-<name>/
  robot-v1/        # or your robot repo
```

In each downstream `Cargo.toml`, add a repo-local `[patch.crates-io]` block
pointing at the sibling paths while iterating. Retire the patches before
opening the PR.

## Commit conventions

- **Conventional Commits.** `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`,
  `test:`. The release tooling (`release-plz`) reads these to compute
  versions and write changelogs — get the prefix right.
- **DCO sign-off** on every commit (`Signed-off-by:`). See
  [governance/CONTRIBUTING.md](../governance/CONTRIBUTING.md).
- **One coherent change per PR.** Mixing a contract update with an unrelated
  cleanup makes review and revert harder.

## Validation expectations

A change to a `phoxal-runtime-<name>-api` payload **lands with**:

- a typed contract test (decode round-trip, including the actionable error
  fields when the schema mismatches);
- the Webots scenario gate that exercises the new payload (or the existing
  gate updated);
- a `README.md` update in the owning crate.

A change to schema crates (`phoxal-core-robot`, `phoxal-core-structure`,
`phoxal-core-component`) lands with parse-round-trip tests and
`deny_unknown_fields` coverage.

A change to the CLI lands with the resolver test that demonstrates the new
behavior end-to-end (lockfile pin, validate pass, simulate dry-run) and a
clean-room run from a fresh user-robot repo.

## When a change requires an architecture decision

Open an issue under the relevant repo's tracker. If the change spans
multiple repos, file it under
[`phoxal/robot-framework`](https://github.com/phoxal/robot-framework/issues)
(the cross-repo control plane) and link affected repos in the body.

Use [Phoxal Kanban](https://github.com/orgs/phoxal/projects/1) to track
status. Issues land on the board; decisions land in code + docs.

## What not to do

- **Don't introduce a new `phoxal-*-api` crate without a clear owning
  runtime.** Operator tools and the CLI consume existing api crates; they
  do not own their own contracts.
- **Don't add a `RuntimeDescriptor`, `--phoxal-describe`, or any runtime
  introspection protocol.** Topology is the CLI's static knowledge.
- **Don't synthesize per-runtime config files.** Runtimes read the resolved
  `robot.yaml` directly.
- **Don't add backwards-compatibility shims for in-flight contracts.** This
  is a pre-1.0 project; breaking changes are loud, not quiet.
- **Don't reach across the engine ↔ CLI boundary.** Resolver, lockfile,
  command implementations are CLI-only; `Runtime` trait, bootstrap, bus
  integration are engine-only. The schema crates are shared between them.

## Repo-local AGENTS.md

Each repo carries its own `AGENTS.md` with implementation-specific rules
(test fixtures, build hooks, local CI invocations). The org-level
[AI_ASSISTANT_GUIDE.md](AI_ASSISTANT_GUIDE.md) is the cross-repo guide;
the repo-local `AGENTS.md` wins for that repo's specifics.

## License

Every PR — whether to a `phoxal/*` repo or a user runtime scaffolded by
`phoxal-cli create` — is contributed under **AGPL-3.0-only**. See
[governance/LICENSE_POLICY.md](../governance/LICENSE_POLICY.md).
