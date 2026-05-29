# Contributing to Phoxal

Thank you for your interest. Phoxal is pre-1.0 and building in public; the
contribution surface is still tightening, so please read this before opening
a PR.

## Where to file

Issues and PRs live in the repository that owns the concern your change
touches. See [docs/REPOSITORY_MAP.md](../docs/REPOSITORY_MAP.md) and
[docs/DEVELOPMENT_MODEL.md](../docs/DEVELOPMENT_MODEL.md) for the
where-does-it-belong rules.

Cross-repo planning and coordination happens on the [Phoxal
Kanban](https://github.com/orgs/phoxal/projects/1).

## Before opening a PR

1. **Open an issue first** if the change is more than a small fix. We'd
   rather discuss approach before code lands.
2. **Read the org-level docs.** [VISION.md](../docs/VISION.md),
   [ARCHITECTURE.md](../docs/ARCHITECTURE.md), and the relevant repo's
   `AGENTS.md`. We try to keep these honest; if something looks wrong,
   please flag it.
3. **Match the existing patterns.** Phoxal optimizes for consistency over
   per-PR novelty.
4. **Cover the change with tests** — see "Required test coverage" in
   [docs/AI_ASSISTANT_GUIDE.md](../docs/AI_ASSISTANT_GUIDE.md).

## Commit conventions

- **Conventional Commits.** `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`,
  `test:`, `perf:`, `ci:`. The release tooling (`release-plz`) reads these
  to compute versions and generate changelogs.
- **One coherent change per PR.** Reviewers should be able to understand
  the PR's purpose in one sentence.
- **Signed off** per the Developer Certificate of Origin (DCO):

  ```
  Signed-off-by: Your Name <you@example.com>
  ```

  Use `git commit -s` to add the sign-off automatically.

By signing off, you certify the DCO terms at <https://developercertificate.org/>.
In short: you wrote the change yourself, or you have the right to submit it
under the same license as the rest of the file (AGPL-3.0-only). DCO sign-off
is required on every commit.

The Phoxal team may revisit DCO vs CLA before accepting external
contributions at scale, in line with the policy work tracked under
[phoxal/.github#7](https://github.com/phoxal/.github/issues/7).

## License

All contributions to Phoxal repositories are submitted under
**AGPL-3.0-only**. See [LICENSE_POLICY.md](LICENSE_POLICY.md).

Generated scaffolds from `phoxal-cli create` default to AGPL-3.0-only as
well.

## Code of Conduct

Be respectful. Disagree about technical decisions, not about people. Don't
make Phoxal a place people regret showing up to.

If something goes wrong, contact the maintainers via the contact path in
the relevant repo (typically `security@phoxal.com` works for sensitive
matters even though it's primarily a security mailbox).

## What kinds of changes are easiest to land

- **Bug fixes** with a regression test.
- **Documentation fixes** — typos, stale links, broken examples.
- **New Webots scenario coverage** for an existing platform runtime.
- **New catalog component** (`phoxal/component-<name>`) following the
  established repo shape.

## What requires more discussion first

- Anything that adds a new `phoxal-*-api` crate or a new platform runtime.
- Changes to authoring schemas (`robot.yaml`, `structure.urdf`,
  `component.yaml`).
- Changes to the CLI's resolver, lockfile, or
  `PlatformRuntimeCatalog`.
- Changes to the trust model (signing, allowlist, lockfile semantics).

For these, open an issue with the proposed shape before writing code.

## Security

Don't report security issues in public issues. See
[SECURITY.md](SECURITY.md).
