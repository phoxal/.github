# CODEX.md — Codex / `codex exec` entry point

You are Codex, working in a Phoxal repository.

**Read these in order:**

1. [`docs/AI_ASSISTANT_GUIDE.md`](../docs/AI_ASSISTANT_GUIDE.md) — the
   canonical, cross-repo rules for any AI agent in any Phoxal repo.
2. The local `AGENTS.md` at the root of this repository — that repo's
   specific rules.
3. The local `README.md` for the crate / sub-tree you're editing.

The org-level guide is the rule unless the repo-local `AGENTS.md` overrides
it for that repo's specifics.

## Codex-specific notes

- **You are typically the writer agent in a reviewer/writer split.** A
  Claude-side reviewer drafts the prompt and reviews your output. Trust
  the prompt's acceptance criteria — the reviewer's role is to verify, not
  to retype your work.
- **Small, well-scoped runs.** A focused prompt with a verifiable
  deliverable beats one mega-prompt that produces an opaque pile of
  changes. The bottleneck is correctness, not run count.
- **Stay within the dispatched scope.** If you discover the change needs
  to spread to a different repo, **stop and report** — cross-repo
  coordination belongs to the reviewer, not to a single codex run.
- **Trust the underlying file contents** when the prompt cites
  file:line locations.

## What not to do

- Don't introduce backwards-compat shims for in-flight v0.x contracts.
- Don't synthesize per-runtime config files.
- Don't add runtime introspection protocols.
- Don't expand scope beyond what the prompt acceptance criteria require.

See the canonical guide for the full list.
