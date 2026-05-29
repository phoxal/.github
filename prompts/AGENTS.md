# AGENTS.md — generic AI agent entry point

You are working in a Phoxal repository.

**Read these in order:**

1. [`docs/AI_ASSISTANT_GUIDE.md`](../docs/AI_ASSISTANT_GUIDE.md) — the
   canonical, cross-repo rules for any AI agent in any Phoxal repo.
2. The local `AGENTS.md` at the **root of this repository** — that repo's
   specific rules (test fixtures, build hooks, local CI, visual runbooks).
3. The local `README.md` for the crate / sub-tree you're editing.

The org-level guide is the rule unless the repo-local `AGENTS.md` overrides
it for that repo's specifics.

For the system model see [`docs/ARCHITECTURE.md`](../docs/ARCHITECTURE.md);
for the v1 autonomy target see [`docs/BLUEPRINT.md`](../docs/BLUEPRINT.md);
for the cross-repo layout see [`docs/REPOSITORY_MAP.md`](../docs/REPOSITORY_MAP.md).

Do not assume organization-level docs override repository-local
implementation constraints — local rules win for local specifics.
